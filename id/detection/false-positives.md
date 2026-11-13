# Tuning & False Positive

False positive adalah alert yang fire pada aktivitas yang sah, dan cukup banyak false positive akan menghancurkan kredibilitas SOC lebih cepat daripada detection yang terlewat mana pun. Tuning adalah pekerjaan berkelanjutan untuk menaikkan precision tanpa menurunkan recall: menghapus noise sambil menjaga serangan nyata tetap terlihat. Halaman ini membahas alur kerja tuning, penyebab noise yang umum, dan cara menjaga severity serta baseline tetap jujur.

## Mengapa False Positive Menghancurkan Kredibilitas

Volume alert tumbuh lebih cepat daripada jumlah analyst. Jika sebuah rule fire ratusan kali sehari dan hanya sangat jarang menggambarkan sesuatu yang nyata, analyst belajar untuk mengabaikannya sekilas, dan pada hari ketika hal itu penting, alert itu ikut diabaikan. Inilah alert fatigue: respons melambat, investigasi menjadi seremonial, dan heuristik pengabaian diam-diam ("selalu host itu") menggantikan analisis yang sebenarnya. Angkanya kurang penting dibanding polanya: noise berlipat ganda, kepercayaan memudar, dan nilai deteksi ikut menurun.

## Precision dan Recall dengan Sederhana

| Ukuran | Pertanyaan yang dijawab | Rumus | Analogi jaring ikan |
|---|---|---|---|
| Precision | Dari alert yang kami hasilkan, berapa yang nyata? | `TP / (TP + FP)` | Jaring yang terlalu lebar menangkap bycatch alih-alih ikan |
| Recall | Dari serangan nyata, berapa yang kami beri alert? | `TP / (TP + FN)` | Jaring yang terlalu kecil membiarkan ikan lewat |

Men-tune sebuah rule untuk menghentikan false positive biasanya berbiaya sebagian recall. Karena itu setiap keputusan pengencangan seharusnya bertanya: serangan nyata apa yang kini bisa terlewat oleh rule ini?

## Alur Kerja Tuning

1. **Triase false positive.** Konfirmasi bahwa itu sah, rekam buktinya (host, akun, proses induk, timestamp), dan beri tag di antrean alert.
2. **Identifikasi penyebabnya.** Apakah sumbernya berisik, polanya terlalu luas, konteksnya hilang, atau parser-nya rusak? Lihat tabel di bawah.
3. **Pilih perbaikan terkecil.** Kencangkan condition, tambahkan exception yang berlingkup sempit, naikkan threshold, atau perbaiki data-nya alih-alih menonaktifkan rule.
4. **Uji ulang.** Jalankan query hasil revisi atas 7 sampai 30 hari log historis dan bandingkan jumlah match serta sampelnya sebelum dan sesudah.
5. **Dokumentasikan.** Catat perubahan, alasannya, pemiliknya, dan tanggal kedaluwarsa exception apa pun dalam version control bersama rule-nya.
6. **Pantau.** Amati rule selama dua minggu; jika volume melonjak lagi, lingkungan telah berubah dan siklus dimulai dari awal.

## Penyebab False Positive yang Umum

| Penyebab | Gejala | Perbaikan umum |
|---|---|---|
| Sumber berisik | Sebagian besar hit berasal dari beberapa host yang dikenal (backup, build, tool deployment) | Batasi cakupan rule atau tambahkan exception terdokumentasi untuk aset-aset tersebut |
| Pola terlalu luas | `\|contains: '-enc'` juga cocok dengan `-Encoding` dan path seperti `C:\encrypted\` | Gunakan `\|endswith`, wajibkan dua field yang cocok, atau gunakan `\|re` dengan anchor |
| Konteks hilang | Tidak bisa membedakan akun sistem dari user, atau service dari penggunaan interaktif | Perkaya data source (user, proses induk, peran host) |
| Parser rusak | Field kosong, terpecah, atau salah nama; rule match pada sampah atau tidak sama sekali | Perbaiki pemetaan field dan validasi ulang terhadap sampel yang diketahui |
| Aktivitas admin yang sah | Script helpdesk atau admin berulang kali memicu rule | Exception sempit dengan pemilik dan masa berlaku, bukan pernah menghapus rule |
| Threshold terlalu rendah | Event `>0` fire pada keseharian harian yang normal | Baseline volume normal dan naikkan threshold di atasnya |

## Exception yang Tepat

Exception mengecualikan sumber tertentu dan beralasan dari sebuah rule. Exception tidak pernah berarti "matikan rule-nya":

```yaml
detection:
  selection:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|contains:
      - 'Invoke-WebRequest'
      - 'DownloadString'
  filter_backup_agent:
    ParentImage|endswith: '\backupagent.exe'
  condition: selection and not filter_backup_agent
```

Setiap exception harus berlingkup jelas (proses, path, akun, atau grup host yang tepat), terdokumentasi (justifikasi, tiket, pemilik), berbatas waktu (tanggal kedaluwarsa atau tinjauan), dan ditinjau secara berkala. Sediakan sebuah register:

| Exception | Rule | Cakupan | Justifikasi | Pemilik | Kedaluwarsa | Tinjauan terakhir |
|---|---|---|---|---|---|---|
| EX-014 | PowerShell download cradle | Parent `backupagent.exe` di `srv-backup*` | Agen vendor memeriksa pembaruan | SOC lead | 2026/07/01 | 2026/04/01 |

## Baselining

Anda tidak bisa menyebut sebuah aktivitas tidak normal tanpa mengetahui seperti apa normalnya di lingkungan Anda sendiri, bukan di lingkungan orang lain. Baseline diukur selama jendela yang representatif (biasanya 2 sampai 4 minggu) dan disegarkan setiap kuartal.

| Metrik | Cara mem-baseline | Yang dideteksinya |
|---|---|---|
| Proses per host per jam | Rata-rata 30 hari dan daftar top-N per peran host | Binary yang jarang atau belum pernah terlihat sebelumnya |
| Volume login per akun (tipe 4624) | Median harian dan host sumber yang biasa | Penggunaan kredensial dari lokasi baru |
| Volume dan panjang query DNS | Distribusi 14 hari per segmen jaringan | Tunneling dan eksfiltrasi low-and-slow |
| Service baru yang terpasang per minggu | Hitungan selama 30 hari | Persistence di luar jalur (out-of-band) |
| Pangsa penggunaan PowerShell di endpoint | Persentase host yang menjalankannya dalam 30 hari | Aktivitas living-off-the-land di mesin yang janggal |

## Kalibrasi Severity

Aturan praktis: `severity = impact × confidence`. Impact adalah kerugian jika perilaku itu berbahaya; confidence adalah seberapa yakin Anda bahwa itu berbahaya.

| Severity | Kapan digunakan | Contoh | Respons yang diharapkan |
|---|---|---|---|
| High | Impact tinggi dan confidence tinggi | Akses memori LSASS oleh binary tidak bertanda tangan | Triage segera, kemungkinan eskalasi |
| Medium | Perilaku mencurigakan, confidence belum terbukti | PowerShell download cradle dari workstation kantor | Review analyst dalam shift yang sama |
| Low | Sinyal lemah, berguna dalam agregasi atau hunt | Satu nilai autorun baru yang ditetapkan user standar | Review dashboard atau pemrosesan batch |

Severity harus dikalibrasi terhadap rasio alert-ke-insiden Anda: jika setiap rule berseverity high, tidak ada yang benar-benar high.

## Rasio Alert-ke-Insiden

Hitung alert yang dihasilkan dan insiden terkonfirmasi selama suatu periode, lalu bagi keduanya. Rasio dalam ribuan berarti noise yang mengatur tempo; rasio dalam satu digit mungkin berarti Anda kurang men-triage atau kurang mendeteksi. Benchmark berbeda antar lingkungan, jadi lacak tren Anda sendiri seiring waktu: tujuannya adalah rasio yang menurun ketika tuning membaik, sementara jumlah insiden tetap datar atau naik.

## Contoh Terperinci: Rule yang Fire pada Segalanya

Versi pertama hanya mensyaratkan proses PowerShell dengan `-enc` di mana pun dalam command line-nya:

```yaml
detection:
  selection:
    Image|endswith: '\powershell.exe'
    CommandLine|contains: '-enc'
  condition: selection
```

Rule ini menghasilkan sekitar 400 alert per hari. Penyebabnya adalah substring yang terlalu luas: `-enc` cocok dengan `-Encoding UTF8` dalam script yang sah, nama direktori seperti `C:\share\encoded\`, dan switch yang tidak berhubungan. Setelah triage, rule ditulis ulang:

```yaml
detection:
  selection:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|contains: '-EncodedCommand'
  selection_short:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|re: '(?i)-enc\s+[A-Za-z0-9+/=]{20,}'
  filter_deploy_account:
    User: 'svc_deploy'
  condition: (selection or selection_short) and not filter_deploy_account
```

Kemudian: jalankan ulang kedua versi atas 30 hari data historis, ambil sampel 20 match dari set hasil masing-masing, konfirmasi bahwa akun deployment admin yang diketahui dikecualikan melalui exception yang terdokumentasi, dan catat jumlah match sebelum/sesudah di pesan commit. Dialek regular expression bergantung pada backend, jadi validasikan di SIEM Anda sebelum men-deploy.

## Tuning Tidak Pernah Selesai

Aplikasi berubah, perangkat lunak baru di-deploy, skema log bergeser, dan penyerang beradaptasi. Rule yang sudah di-tune adalah sebuah cuplikan (snapshot), bukan garis finis: jadwalkan tinjauan rule berkala, pantau volume alert per rule, dan perlakukan setiap laporan false positive sebagai data untuk iterasi berikutnya.

## Lihat juga

- [Siklus Deteksi](detection-lifecycle.md)
- [Menulis Aturan Sigma](sigma-rules.md)
- [Triage Alert](../response/triage-alerts.md)
- [Proses Incident Response](../response/incident-response.md)
- [Dasar SIEM](../fundamentals/siem-basics.md)
