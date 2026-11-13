# Dasar Security Operations

Security operations adalah pekerjaan sehari-hari mengawasi sistem, memutuskan sinyal mana yang penting, dan merespons sebelum masalah kecil berubah menjadi insiden. Security Operations Center (SOC) adalah tim dan fungsi yang menjalankannya, dan kinerjanya adalah fondasi bagi sebagian besar karier blue team. Halaman ini membahas struktur SOC, siklus hidup sebuah alert, metrik yang dilacak para pemimpin, dan realitas kerja shift.

## Apa Itu SOC

SOC adalah fungsi yang terorganisasi, artinya sebuah tim, kumpulan proses, dan kumpulan alat, yang memantau sistem sebuah organisasi serta mendeteksi, menyelidiki, dan merespons peristiwa keamanan. SOC bukanlah ruangan berdinding gelap dengan layar-layar di dinding; melainkan model operasi yang berulang dan bisa diandalkan pada pukul 3 pagi sama seperti pada siang hari.

Tumpukan teknologi yang umum sederhana untuk disebut: **SIEM + EDR + tiket**. SIEM mengumpulkan dan mengorelasikan log, EDR memberikan visibilitas endpoint serta aksi kontainment, dan sistem tiket mencatat setiap alert, keputusan, dan tindakan agar pekerjaan tetap bertahan saat berganti shift.

## Tingkatan SOC

| Tingkat | Fokus | Pekerjaan umum |
| --- | --- | --- |
| L1 — Triage | Pandangan pertama | Meninjau alert, mengumpulkan konteks dasar (pengguna, host, reputasi), menutup peristiwa yang sudah diketahui benign, menaikkan true positive dengan catatan |
| L2 — Investigasi | Kedalaman | Berpindah (pivot) di antara SIEM dan EDR, menentukan lingkup, memutuskan kontainment, menangani insiden sampai selesai |
| L3 — Hunting dan engineering | Peningkatan proaktif | Threat hunting, membangun dan menyetel aturan deteksi, analisis akar masalah, perangkat dan otomasi |
| Manajemen | Arah | Pengaturan staf dan shift, tingkat layanan, metrik dan pelaporan, jalur eskalasi, koordinasi antar tim |

Tingkatan menggambarkan tanggung jawab, bukan gerbang senioritas. Di tim kecil, satu analis mungkin melakukan triage, investigasi, dan menulis aturan deteksi dalam shift yang sama.

## Siklus Hidup Alert

Setiap alert mengikuti jalur yang sama, apa pun alat atau perusahaannya:

1. **Detect** — sebuah aturan atau ambang terpicu pada telemetri dan membuat sebuah alert.
2. **Triage** — analis memastikan alert itu nyata, relevan, dan sesuai lingkup; peristiwa yang sudah diketahui benign ditutup dengan alasan.
3. **Investigate** — perluas ke luar: host yang terdampak, akun, peristiwa terkait, dan apakah sistem lain turut terlibat.
4. **Contain** — hentikan kerusakan: isolasi host, nonaktifkan akun, blokir indikator, cabut sesi.
5. **Document** — catat apa yang terjadi, apa yang dilakukan, dan apa yang harus berubah agar alert berikutnya lebih cepat ditangani.

Melewatkan dokumentasi adalah jalan pintas yang paling umum, dan justru itu yang paling menyakitkan saat insiden shift malam berikutnya datang.

## Metrik Utama

| Metrik | Artinya | Mengapa penting |
| --- | --- | --- |
| MTTD (mean time to detect) | Waktu dari tindakan pertama penyerang hingga alert Anda | Menunjukkan seberapa besar serangan yang Anda lihat, dan seberapa dini |
| MTTR (mean time to respond or resolve) | Waktu dari deteksi hingga kontainment atau penutupan | Menunjukkan seberapa cepat tim bertindak setelah sesuatu terpicu |
| Volume alert | Alert per shift atau per hari | Menentukan kebutuhan staf; lonjakan mendadak biasanya berarti peristiwa nyata atau aturan yang rusak |
| Rasio false positive | Bagian alert yang ternyata bukan insiden nyata | Pendorong utama alert fatigue; rasio tinggi melatih analis untuk meremehkan alert |

Organisasi mendefinisikan batas MTTD dan MTTR secara berbeda, misalnya tindakan pertama versus log pertama, atau deteksi versus kontainment versus penutupan. Sepakati definisinya terlebih dahulu sebelum membandingkan angka.

## Realitas On-Call dan Kerja Shift

Deteksi tidak berhenti semalaman, sehingga SOC menjalankan shift dengan cakupan akhir pekan dan hari libur. Dalam praktiknya ini berarti serah terima di batas setiap shift, rotasi on-call untuk eskalasi di luar jam kerja, dan jalur tertulis saat satu analis kewalahan.

Dua hal membuat kerja shift berkelanjutan: catatan serah terima yang membuat konteks bisa dipindahkan, dan aturan yang jelas kapan seseorang harus dibangunkan. Keduanya adalah masalah proses, bukan masalah kepribadian, dan layak diperbaiki sejak awal.

### Yang wajib ada dalam serah terima shift

- Insiden yang masih terbuka dan status kontainmentnya saat ini.
- Host yang masih diisolasi atau akun yang masih dinonaktifkan, serta siapa yang bertugas melepasnya.
- Aturan yang bising atau rusak dan membanjiri shift dengan alert.
- Gangguan perangkat: collector yang berhenti, connector yang kedaluwarsa.
- Tindakan yang menunggu dengan tenggat waktu, misalnya pengambilan citra forensik yang sudah dijadwalkan.
- Di mana analis sebelumnya meninggalkan catatannya, di tiket dan bukan hanya di chat.

## Runbook vs Playbook

- **Runbook** — instruksi taktis per alert: konsol mana yang harus dibuka, field mana yang harus diperiksa, kapan harus menaikkan. Contoh: "EDR alert tipe X: periksa proses parent; jika parent-nya Office, eskalasi ke L2."
- **Playbook** — prosedur yang lebih luas untuk satu tipe insiden, mencakup peran, opsi kontainment, komunikasi, dan penanganan bukti. Contoh: "Diduga ransomware: isolasi, beri tahu eksekutif, amankan bukti forensik, ikuti urutan pemulihan."

Runbook memberi tahu analis apa yang harus dilakukan dengan satu alert. Playbook memberi tahu seluruh tim apa yang harus dilakukan dengan satu insiden. Keduanya harus cukup singkat untuk dipakai di bawah tekanan.

## Alert Fatigue

Masalah inti dari pekerjaan SOC bukanlah kekurangan alert; melainkan terlalu banyak alert yang tidak penting. Ketika sebagian besar antrean adalah kebisingan, insiden nyata mendapatkan perlakuan sekali baca yang sama seperti kebisingan, dan biayanya muncul sebagai deteksi yang terlewat.

Pembaikannya adalah sebuah siklus, bukan pembelian: ukur aturan mana yang menghasilkan false positive terbanyak, setel atau nonaktifkan aturan tersebut, tambahkan penekanan (suppression) untuk aktivitas yang sudah diketahui baik, lalu ukur ulang. Analis yang bisa mengajukan saran penyetelan dalam satu klik sedang memberi SOC aliran data paling berharganya.

## Contoh Terinci: Perjalanan Sebuah Alert

Skenario: sebuah alert EDR terpicu di workstation keuangan, dan alert tersebut menempuh perjalanan dari deteksi hingga penutupan.

```
09:12  DETECT   EDR rule: "suspicious PowerShell spawned by Word" on WRK-117
09:14  TRIAGE   L1 checks process tree: winword.exe -> powershell.exe -enc <blob>
                 User: j.mayer   Host: patch level current   No prior alerts
                 Verdict: not a known software deploy -> ESCALATE to L2
09:21  INVEST   L2 pivots in SIEM: same encoded command on 1 more host (WRK-142)
                 Outbound connection to rare-domain.example, first seen yesterday
09:34  CONTAIN  Both hosts isolated in EDR; domain added to proxy and DNS
                 blocklists; affected user sessions revoked and password reset
09:50  SCOPE    Fleet hunt for the parent-child pattern: no further hits
10:40  DOCUMENT Ticket filled with timeline, indicators, and actions taken;
                 new detection idea filed for the engineering backlog
11:05  CLOSE    Alert closed as a confirmed incident; post-incident review set
```

Timeline-nya adalah intinya: setiap langkah punya pemilik, sebuah keputusan, dan catatan tertulis. Alert tanpa ketiganya hanyalah notifikasi.

## Lanskap Perangkat

Tiga kategori alat menopang sebagian besar alur kerja SOC: **SIEM** untuk mengsentralisasi dan mengorelasikan log, **EDR** untuk telemetri endpoint dan kontainment, serta **tiket** sebagai sistem catatan resmi pekerjaan, dengan sumber enrichment seperti lookup threat-intel, DNS, dan log proxy yang dicolokkan ke dalam siklus yang sama.

## Lihat juga

- [Apa Itu Blue Team](what-is-blue-team.md)
- [Triage Alert](../response/triage-alerts.md)
- [Playbook & Checklist](../response/playbooks.md)
- [Penyetelan & False Positive](../detection/false-positives.md)
