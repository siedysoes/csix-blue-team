# Threat Hunting

Threat hunting (perburuan ancaman) adalah pencarian aktif yang berbasis hypothesis terhadap aktivitas penyerang yang tidak menghasilkan alert. Karena tidak ada program detection yang mencakup semuanya, hunting adalah cara Anda menemukan titik buta alih-alih menunggu titik buta itu menemukan Anda. Hunting paling berharga di tim yang matang: semakin baik alert Anda, semakin berharga pertanyaan yang belum pernah diajukan siapa pun.

## Apa Itu Hunting (dan Bukan)

Hunting dimulai dari pertanyaan tentang perilaku penyerang, bukan dari dashboard. Bukan menunggu alert datang, dan bukan pula grepping sembarangan tanpa pertanyaan yang dinyatakan. Hunt adalah investigasi dengan batas waktu (time-boxed) yang memiliki hypothesis tertulis, data source yang terdefinisi, dan hasil terdokumentasi yang diberi kembali ke deteksi.

## Hunting Maturity Model

Berdasarkan SANS Hunting Maturity Model (verifikasi detailnya jika Anda memformalkan program Anda sendiri):

| Level | Nama | Karakteristik |
|---|---|---|
| H0 | Minimal | Data tersimpan di silo; pekerjaan hanya digerakkan alert; tidak ada pencarian proaktif |
| H1 | Minimal automated | Sebagian telemetri terpusat; hunt sesekali; minim pengulangan dan dokumentasi |
| H2 | Programmatic | Hunt dijadwalkan, hypothesis dilacak, hasil secara andal menjadi rule baru |
| H3 | Optimized | Telemetri kaya plus otomasi dan threat intelligence; hunt diukur dan ditingkatkan |

Sebagian besar tim harus membidik H2: ritme hunt berkala dengan hypothesis yang tercatat lebih penting daripada tooling canggih.

## Siklus Hunt

| Fase | Aksi | Keluaran |
|---|---|---|
| 1. Hypothesis | Pilih sebuah technique atau perilaku; tentukan apa yang akan memastikan atau menyangkalnya | Hypothesis tertulis dan dapat diuji |
| 2. Kumpulkan dan siapkan | Konfirmasi data source, jendela retensi, dan rentang waktu | Data siap query |
| 3. Selidiki | Jalankan query, pivot antar endpoint, perkaya dengan konteks aset | Daftar temuan beserta bukti |
| 4. Iterasi | Persempit atau ganti hypothesis berdasarkan apa yang ditunjukkan data | Hypothesis baru atau yang disempurnakan |
| 5. Dokumentasikan | Catat query, temuan, dan gap | Laporan hunt plus kemungkinan detection rule baru |

## Menulis Hypothesis

Hypothesis yang lemah tidak bisa gagal, jadi tidak bisa mengajarkan apa pun pada Anda. Hypothesis yang kuat menyebut sebuah perilaku, sebuah cakupan, dan bukti yang bisa menyelesaikannya.

| Hypothesis lemah | Hypothesis kuat | Mengapa lebih kuat |
|---|---|---|
| "Ada yang mencurigakan di PowerShell" | "Adversary yang hidup dari alat bawaan sistem (living off the land) akan menggunakan `certutil.exe` untuk mengunduh file di fleet Windows kami" | Menyebut sebuah binary, sebuah perilaku, dan sebuah cakupan |
| "Mungkin ada lateral movement" | "Penyerang dengan kredensial valid akan menggunakan WMI untuk menjalankan command di beberapa server, terlihat sebagai `wmiprvse.exe` yang melahirkan `cmd.exe`" | Menyebut technique-nya, hubungan parent-child, dan ambang batas |
| "Periksa DNS untuk apa pun yang aneh" | "DNS tunneling akan muncul sebagai query berulang ke satu domain dengan subdomain lebih panjang dari 40 karakter" | Mendefinisikan pola yang dapat diamati dan terukur |
| "Cari akun baru" | "Persistence akan dibuat melalui akun lokal yang ditambahkan di luar jendela perubahan pada server" | Mengaitkan perilaku dengan sebuah proses dan cakupan waktu |

Kebiasaan yang baik: tulis hypothesis-nya, lalu tuliskan dalam satu kalimat bukti apa yang akan membuktikannya salah, sebelum Anda menjalankan satu query pun.

## Hunt Berbasis ATT&CK

Gunakan coverage matrix Anda (lihat [Siklus Deteksi](detection-lifecycle.md)) sebagai daftar backlogged hunt. Technique yang tidak punya rule, tidak punya telemetri, atau punya rule yang belum diuji adalah kandidat hunt; hunt memberi tahu Anda apakah perilaku itu muncul di lingkungan Anda dan data apa yang Anda butuhkan untuk mendeteksinya. Saat menghunt sebuah technique, verifikasi ID-nya terlebih dahulu di attack.mitre.org, lalu tanyakan: apakah kami bahkan mencatat jejak yang ditinggalkan technique ini?

## Ide Hunt Umum

| Ide hunt | Referensi ATT&CK (verifikasi) | Data source | Sudut query pertama |
|---|---|---|---|
| Penggunaan PowerShell mencurigakan | T1059.001 | Pembuatan proses dengan command line (Sysmon 1 / 4688 / EDR) | Command line yang mengandung `-EncodedCommand`, `FromBase64String`, `IEX` |
| Pembuatan service tidak biasa | T1543.003 | System 7045 atau Security 4697 | Service yang dibuat di luar jendela perubahan atau oleh akun non-admin |
| Lateral movement via WMI | T1047 | Pembuatan proses, log WMI-Activity | Proses yang dilahirkan oleh `wmiprvse.exe` |
| Anomali DNS dan tunneling | T1071.004 | Log query DNS | Subdomain panjang, domain jarang, volume query tinggi per host |
| Lateral movement RDP | T1021.001 | Security 4624 dengan LogonType 10 | Akun yang login ke banyak host dalam jendela waktu singkat |

## Tooling

Hunting sebagian besar adalah pekerjaan query atas telemetri yang sudah Anda miliki. SPL untuk Splunk dan KQL untuk Microsoft Sentinel atau Defender adalah bahasa SIEM yang umum; konsol EDR mengekspos endpoint query bergaya yang sama; osquery mengubah endpoint menjadi tabel SQL; dan Velociraptor menambahkan forensik hunting seluruh fleet dengan VQL. Contoh sudut KQL untuk hunt PowerShell:

```kql
DeviceProcessEvents
| where FileName in~ ("powershell.exe", "pwsh.exe")
| where ProcessCommandLine has_any ("Invoke-WebRequest", "DownloadString", "-EncodedCommand")
| summarize Hits = count() by AccountName, DeviceName, bin(Timestamp, 1h)
| where Hits > 5
```

Dan sudut osquery yang aman untuk inspeksi lokal mesin Anda sendiri:

```sql
SELECT p.pid, p.path, p.cmdline, u.username
FROM processes p JOIN users u ON p.uid = u.uid
WHERE p.cmdline LIKE '%DownloadString%' OR p.cmdline LIKE '%-EncodedCommand%';
```

## Ketika Anda Menemukan Sesuatu

Menutup lingkaran adalah inti dari hunting. Validasikan temuannya, tentukan cakupannya ke seluruh fleet, dan jika berbahaya, serahkan ke proses incident response lengkap dengan bukti Anda. Kemudian tulis atau tingkatkan detection rule agar perilaku yang sama alert pada berikutnya, dan catat telemetri yang hilang sebagai gap data source. Hunt yang berakhir hanya dengan pesan Slack baru menyelesaikan separuh pekerjaan.

## Dokumentasi dan Berbagi

Laporan hunt yang singkat dan konsisten menjaga pengetahuan tetap di tim, bukan di riwayat shell satu analyst:

- Judul, tanggal, hunter, dan batas waktu
- Hypothesis, termasuk apa yang akan menyangkalnya
- Cakupan: host, akun, rentang waktu, data source yang digunakan
- Query yang dijalankan, bisa disalin-tempel agar hunt berikutnya dapat mengulanginya
- Temuan beserta bukti (host, user, timestamp, artefak)
- Detection yang dibuat atau diperbarui, serta technique ATT&CK yang tercakup
- Gap yang ditemukan (log hilang, titik buta) dan langkah berikutnya

## Lihat juga

- [Siklus Deteksi](detection-lifecycle.md)
- [Menulis Aturan Sigma](sigma-rules.md)
- [Tuning & False Positive](false-positives.md)
- [Triage Alert](../response/triage-alerts.md)
- [Defense Labs & Ranges](../professional/defense-labs.md)
