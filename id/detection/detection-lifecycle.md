# Siklus Deteksi

Sebuah detection adalah produk, bukan sekadar skrip sekali pakai: detection dibayangkan, dibangun, dirilis, digunakan oleh analyst, dan akhirnya diubah atau dipensiunkan. Memperlakukan rule sebagai aset yang hidup adalah pembeda antara program detection yang meningkat setiap kuartal dengan program yang perlahan membusuk menjadi noise. Halaman ini menelusuri siklus penuh, dari hypothesis hingga pensiun, serta metrik yang menunjukkan apakah detection Anda berfungsi.

## Mengapa "Tulis dan Lupakan" Gagal

- **Noise menumpuk.** Setiap rule yang ditulis tanpa pengujian historis akan fire pada aktivitas yang sah. Segelintir rule yang berisik saja sudah cukup untuk mendorong alert nyata keluar dari layar.
- **Lingkungan berubah (drift).** Aplikasi berubah, agent di-upgrade, field log bergeser atau diganti nama, dan rule yang cocok sempurna di bulan Januari diam-diam tidak cocok apa pun di bulan Juni.
- **Jalur serangan berubah.** Adversary mengadopsi tool dan technique baru; coverage yang lengkap tahun lalu kini punya celah.
- **Kepercayaan tergerus.** Ketika analyst berulang kali melihat alert yang ternyata tidak ada isinya, mereka berhenti menyelidiki dengan cermat. Biayanya dihitung dari insiden nyata yang terlewat.

## Fase-Fase Siklus

| Fase | Yang terjadi | Kriteria keluar |
|---|---|---|
| 1. Hypothesis | Nyatakan perilaku penyerang yang ingin Anda tangkap, dan data yang bisa membuktikannya | Pertanyaan yang dapat diuji, dituliskan |
| 2. Riset dan ATT&CK mapping | Temukan technique-nya, tulisan-tulisan nyata, dan field log yang membukanya | Technique ID dan data source terkonfirmasi |
| 3. Tulis rule | Susun detection (Sigma atau query native) | Rule lolos validasi sintaks |
| 4. Uji terhadap log | Jalankan atas data historis, periksa sampel match secara manual | Known-bad match, known-good tidak |
| 5. Deploy | Konversi, jadwalkan atau stream, arahkan output ke SOC | Alert tiba di antrean lengkap dengan konteks |
| 6. Alert dan triage | Analyst menyelidiki; temuan diberi kembali ke pemilik rule | Runbook tersedia; keputusan dicatat |
| 7. Tune | Perbaiki false positive, sesuaikan severity, tambahkan exception yang berlingkup | Volume alert berkelanjutan |
| 8. Ganti atau tingkatkan | Gantikan rule yang usang, tentukan ulang cakupan setelah perubahan infrastruktur | Alasan didokumentasikan dalam version control |

Fase 6 sampai 8 kembali berputar ke fase 1: alert yang sudah di-triage sering kali menjadi hypothesis berikutnya.

## Dari Hypothesis ke Rule

Hypothesis yang baik menyebut perilaku penyerang dan bukti yang akan ditinggalkannya. Yang buruk mendeskripsikan perasaan ("ada yang terasa janggal").

| Hypothesis | Data yang dibutuhkan | Gagasan rule |
|---|---|---|
| Penyerang akan menggunakan PowerShell terenkripsi untuk menyembunyikan payload | Pembuatan proses dengan command line lengkap (Sysmon 1 atau Security 4688) | Tandai `-EncodedCommand` yang diikuti string base64 panjang |
| Persistence akan ditambahkan melalui key Run | Event set nilai registry (Sysmon 12/13) | Alert atas nilai `Run`/`RunOnce` baru yang ditetapkan oleh akun tidak biasa |
| Pencurian kredensial akan menarget memori LSASS | Event akses proses (Sysmon 10) atau telemetri proses EDR | Alert atas proses jarang yang membuka `lsass.exe` |
| Build agent kami sah menjalankan PowerShell sepanjang hari | Log proses yang sama plus inventaris aset | Rule invers: alert jika PowerShell berjalan di luar host admin yang dikenal |

## Pemetaan ke MITRE ATT&CK

Memetakan setiap rule ke technique ID mengubah tumpukan rule menjadi gambaran coverage yang terukur. **Coverage matrix** mencantumkan technique yang Anda pedulikan serta detection, data source, dan hunt mana yang menyentuhnya; sel kosong adalah titik buta Anda. Verifikasi setiap technique ID terhadap https://attack.mitre.org/ sebelum menerbitkan matrix, karena ID yang diingat dari ingatan mudah salah.

| Technique | Data source | Rule | Status |
|---|---|---|---|
| T1059.001 PowerShell | Pembuatan proses dengan command line | Suspicious PowerShell Download Cradle | Deployed |
| T1547.001 Run key persistence | Registry value set | Registry Persistence via Run Key | Testing |
| T1003.001 LSASS memory access | Sysmon 10 / EDR process access | Suspicious LSASS Access | Draft |
| T1021.001 RDP logons | Security 4624, LogonType 10 | (tidak ada) | Gap: hunt dulu |

Coverage bukan tujuan dengan sendirinya. Satu rule yang di-tune baik dengan telemetri yang baik mengalahkan sepuluh rule yang tidak pernah fire karena data dasarnya tidak ada.

## Detection as Code

Perlakukan rule seperti perangkat lunak: simpan di Git, wajibkan pull request untuk setiap perubahan, dan jalankan pemeriksaan otomatis sebelum merge.

- **Riwayat versi:** setiap perubahan tuning punya penulis, tanggal, dan alasan.
- **Review:** sepasang mata kedua menangkap nama field yang rusak dan condition yang terlalu luas.
- **CI:** validasi sintaks (`sigma check`), lint tag dan ID, dan jalankan query terhadap sampel log known-bad serta known-good yang tersimpan.
- **Deployment:** pipeline mengonversi dan mendorong rule ke SIEM, alih-alih mengedit pencarian di konsol secara manual.

## Atribut Kualitas

| Atribut | Bahasa sederhana | Biaya jika buruk |
|---|---|---|
| Precision | Dari alert yang dihasilkan rule ini, berapa yang nyata (`TP / (TP + FP)`) | Alert fatigue dan hilangnya kredibilitas |
| Recall | Dari serangan nyata jenis ini, berapa yang berhasil ditangkap (`TP / (TP + FN)`) | Intrusi yang senyapan dan berhasil |
| Ketepatan waktu | Jarak antara aksi penyerang dan alert (MTTD) | Waktu lebih banyak bagi penyerang untuk bertindak |
| Stabilitas | Seberapa bervariasinya volume alert dari minggu ke minggu | Tuning darurat yang terus-menerus |
| Konteks | Bukti yang disertakan dalam alert (user, proses induk, peran host) | Triage lambat dan penuh tebakan |

Precision dan recall saling bertukar (trade-off): mengencangkan rule memangkas false positive tetapi juga bisa memangkas detection nyata. Selalu ukur ulang keduanya setelah perubahan tuning.

## Metrik untuk Program Detection

| Metrik | Cara menghitung | Yang diberitahukannya |
|---|---|---|
| Coverage ATT&CK % | Technique dalam cakupan yang punya minimal satu rule teruji, dibagi teknik dalam cakupan | Di mana titik buta berada |
| Rasio alert-ke-insiden | Alert yang dihasilkan dibagi insiden terkonfirmasi | Apakah noise meredam sinyal nyata |
| MTTD | Median waktu dari event ke alert | Seberapa cepat Anda mendeteksi |
| Kesehatan rule | Rule dengan alert nol dalam 30 hari, atau lonjakan mendadak | Pipeline rusak atau perilaku baru |
| False positive rate per rule | Alert FP dibagi total alert, per rule | Rule mana yang harus di-tune lebih dulu |
| Hunt conversion rate | Hunt yang menghasilkan rule baru atau peningkatan | Apakah hunting memberi hasil |

## Contoh Terperinci: Satu Detection dari Ide hingga Rule yang Di-Tune

1. **Hypothesis.** "Adversary akan menggunakan PowerShell untuk mengunduh payload, dan endpoint kami mencatat command line lengkap."
2. **Riset dan pemetaan.** Konfirmasi T1059.001 di attack.mitre.org; periksa 30 hari log proses untuk memastikan field-field itu ada dan terisi.
3. **Menulis.** Susun [Sigma rule](sigma-rules.md) dengan selection `Image|endswith` plus `CommandLine|contains`.
4. **Menguji.** Jalankan query hasil konversi atas 14 hari data historis. Hitung match-nya, lalu baca 20 di antaranya secara manual: harapkan script admin dan software deployer di antara hasilnya.
5. **Deploy.** Konversi ke bahasa query SIEM, jadwalkan, dan arahkan alert ke antrean tuning alih-alih ke halaman on-call. Severity `medium`.
6. **Alert.** Sertakan runbook singkat: apa yang harus diperiksa, kapan harus eskalasi, dan siapa pemilik rule-nya.
7. **Tune.** Setelah satu minggu, satu tool deployment menyumbang sebagian besar hit. Tambahkan exception yang berlingkup sempit untuk proses induk tersebut, dokumentasikan pemilik dan masa berlaku, lalu jalankan ulang pengujian historis untuk mengukur penurunannya.
8. **Meningkatkan.** Sebuah alert memicu hunt untuk downloader lainnya, yang menghasilkan rule kedua untuk LOLBin berbeda. Kedua rule masuk ke matrix ATT&CK; rule yang tidak didukung data apa pun ditandai sebagai gap telemetri, bukan sebagai keberhasilan.

## Lihat juga

- [Menulis Aturan Sigma](sigma-rules.md)
- [Tuning & False Positive](false-positives.md)
- [Threat Hunting](threat-hunting.md)
- [Triage Alert](../response/triage-alerts.md)
- [Dasar Security Operations](../fundamentals/security-operations.md)
