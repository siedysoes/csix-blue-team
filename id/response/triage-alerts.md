# Triage Alert

Triage alert adalah pekerjaan harian untuk memutuskan apakah sebuah deteksi merupakan ancaman nyata, seberapa luas cakupannya, dan apa tindakan selanjutnya. Sebagian besar waktu SOC dihabiskan di sini, dan kualitas keputusan inilah yang menentukan apakah insiden nyata terlewat. Artikel ini menggambarkan alur kerja yang praktis bagi analis yang menangani antrian pertahanan (defensive queue) yang berwenang.

## Realitas Harian Antrian

Antrian tidak pernah kosong. Alert datang dalam gelombang, duplikat berkelompok di sekitar satu aturan yang berisik, dan selalu ada setidaknya satu alert yang lebih ambigu dari yang Anda inginkan. Tugasnya bukan menyelidiki segalanya secara mendalam — melainkan mencapai disposition (keputusan akhir) dengan cepat: eskalasi, tutup, atau tunda, masing-masing disertai catatan yang dapat diikuti analis berikutnya. Kualitas disposition mengalahkan kecepatan mentah; "tutup" yang salah jauh lebih mahal daripada eskalasi yang lambat.

## Pohon Keputusan Triage

```text
[Alert fires]
     |
     v
[Context enrichment]  -- asset criticality, user, IP, process lineage, history
     |
     v
[Is it real?] -- no --> [Close as FALSE POSITIVE: document the WHY]
     |
    yes / suspicious
     v
[Scope]  -- same indicator on other hosts? other accounts? same time window?
     |
     v
[Classify severity]  Critical/High -> escalate to L2/IR now
     |                 Medium       -> escalate with notes, keep monitoring
     |                 Low          -> handle or ticket within SLA
     v
[Act: escalate or close]  -- ALWAYS leave notes: what you checked,
                             what you found, what you did
```

## Sumber Enrichment

| Sumber | Informasi yang diberikan | Contoh pertanyaan |
|---|---|---|
| Kritisitas aset | Dampak bisnis dari host tersebut | Ini domain controller atau kiosk? |
| Departemen / peran pengguna | Apakah pengguna merupakan target yang mungkin | Ini akuntansi, asisten eksekutif, atau akun bersama? |
| Reputasi IP / reverse DNS | Sifat endpoint remote | Penyedia hosting ternama, ISP residensial, atau exit Tor? |
| Lineage proses | Apakah perilaku tersebut masuk akal | Apakah WINWORD.EXE baru saja melahirkan powershell.exe? |
| Alert sebelumnya pada host yang sama | Pola dan riwayat | Apakah host ini sudah menghasilkan tiga alert minggu ini? |

## Contoh Nyata: Alert PowerShell yang Mencurigakan

Alert: PowerShell dijalankan dengan perintah ter-encode di WS-2207 pukul 10:42.

1. **Periksa proses induk.** EDR menunjukkan `WINWORD.EXE` melahirkan `powershell.exe`. Dokumen Word seharusnya tidak melahirkan shell — sudah mencurigakan.
2. **Periksa command line.** Argumen `-enc` membawa sebuah blob base64; tampilan dekode EDR menunjukkan pengunduhan sebuah executable dari host eksternal. Kini kemungkinan besar berbahaya.
3. **Periksa pengguna dan host.** Pengguna adalah akuntan pada sebuah workstation keuangan — target yang masuk akal. Tidak ada alert sebelumnya di host ini, sehingga ini mungkin titik masuk, bukan pengulangan.
4. **Periksa sisi remote.** Reverse DNS dan reputasi pada host pengunduhan menunjukkan penyedia hosting bulletproof dengan kiriman berbahaya baru-baru ini. Reputasi mendukung temuan tersebut tetapi tidak menggantikan langkah 1-3.
5. **Tentukan cakupannya.** Tanyakan SIEM untuk domain, IP, dan hash file yang sama di seluruh armada; periksa apakah ada alert PowerShell-dengan-perintah-encode lain dalam dua jam terakhir. Satu host lagi terkena.
6. **Ambil keputusan.** Berbahaya terkonfirmasi, dua host terdampak → High. Eskalasi ke L2/IR, isolasi kedua host sesuai playbook, dan serahkan catatan yang lengkap.

Contoh catatan eskalasi:

```
Alert: PowerShell encoded command, WS-2207 (Finance), 10:42
Parent: WINWORD.EXE -> powershell.exe -enc <decoded: download of payload>
User: j.doe (Finance). Host had no prior alerts.
Scope query: same hash found on WS-2211 at 10:47. Domain blocked at proxy.
Actions: both hosts isolated per ransomware/malware playbook; passwords pending reset.
Handing to L2 for eradication support. Ticket IR-417.
```

## Menutup False Positive dengan Benar

Dokumentasikan MENGAPA — Anda di masa depan akan berterima kasih, dan begitu juga tim detection engineering. Alasan penutupan yang berguna menyebutkan mekanisme jinak beserta buktinya:

```
Closed: FALSE POSITIVE. powershell.exe invoked by SCCM client
(C:\Windows\CCM\...) on a fixed schedule; binary is vendor-signed;
pattern seen 40x on 12 managed hosts. Suggest narrowing the rule to
non-CCM parents. Feeding to detection tuning backlog.
```

Penutupan yang samar ("terlihat aman", "jinak") memaksa analis berikutnya mengulang pekerjaan dan menyembunyikan peluang penyesuaian. Jangan pernah menutup berdasarkan reputasi saja — IP yang bersih di balik serangan nyata adalah kegagalan deteksi yang paling klasik.

## Kriteria Eskalasi

| Kondisi | Eskalasi ke | Urgensi |
|---|---|---|
| Eksekusi berbahaya terkonfirmasi atau C2 aktif | L2 / IR | Segera |
| Indikator yang sama pada beberapa host atau akun | IR | Segera |
| Akun privilese atau service diduga terkompromi | IR + tim identitas | Segera |
| Akses atau pengunduhan data massal yang anomali | IR + pemilik data | Dalam shift yang sama |
| Mencurigakan tetapi bukti tidak cukup untuk memutuskan | L2 untuk pendapat kedua | Dalam shift |
| Tidak dapat memverifikasi karena tooling atau akses tidak ada | L2 | Jangan menebak |

## Kecepatan vs Kelengkapan

Beri timebox pada alert sederhana (misalnya 5-10 menit untuk keputusan rutin L1) dan eskalasi saat anggaran waktu habis daripada membiarkan alert menggantung. Prioritaskan berdasarkan kritisitas aset dan potensi severity: domain controller lebih tinggi daripada PC ruang rapat. Kedalaman menjadi milik L2 dan IR — tugas Anda adalah disposition yang benar, prioritas yang benar, dan catatan yang lengkap. Jika Anda tidak dapat memutuskan, eskalasi selalu lebih murah daripada penutupan yang salah.

## Kesalahan Triage Umum

| Kesalahan | Konsekuensi | Perbaikan |
|---|---|---|
| Menutup berdasarkan reputasi saja | Serangan di balik IP atau CDN yang "bersih" terlewat | Selalu periksa lineage proses dan command line |
| Mengabaikan cakupan | Satu alert ditutup, sepuluh host terinfeksi lain terlewat | Selalu jalankan satu kueri cakupan seluruh armada |
| Tanpa catatan atau catatan samar | Analis berikutnya mengulang pekerjaan; audit gagal | Catat apa yang Anda periksa dan mengapa Anda memutuskan |
| Memperlakukan setiap alert sebagai kritis | Burnout; insiden nyata tertimbun | Klasifikasikan severity secara konsisten |
| Melakukan pivot tanpa henti pada satu alert | Antrian tumbuh tanpa terkendali | Timebox dan eskalasi saat terhambat |

## Lihat juga

- [Proses Incident Response](incident-response.md)
- [Tuning & False Positive](../detection/false-positives.md)
- [Sumber Log yang Penting](../fundamentals/log-sources.md)
- [Playbook & Checklist](playbooks.md)
