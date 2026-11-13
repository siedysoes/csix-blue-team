# Dasar SIEM

Security Information and Event Management (SIEM) adalah tempat di mana telemetri keamanan sebuah organisasi berkumpul: SIEM mengsentralisarkan log dari banyak sumber, menormalisasikannya ke bentuk yang umum, mengorelasikannya dari waktu ke waktu, dan memunculkan alert ketika pola terlihat salah. Hampir setiap alur kerja SOC melewati SIEM, sehingga mekanismenya penting untuk dipahami bahkan jika Anda tidak pernah mengelolanya. Halaman ini membahas arsitektur, aturan correlation dengan contoh terinci, pilihan platform yang umum, serta faktor-faktor yang menentukan apakah sebuah SIEM membantu atau berubah menjadi arsip log yang mahal.

## Apa yang Dilakukan SIEM

Seluruh produknya dapat direduksi menjadi pipeline empat langkah:

**centralize -> normalize -> correlate -> alert**

1. **Centralize** — menarik peristiwa dari endpoint, server, layanan cloud, dan perangkat ke satu tempat.
2. **Normalize** — mengurai baris mentah menjadi field terstruktur agar logon Windows dan kegagalan SSH Linux bisa dibandingkan.
3. **Correlate** — mengevaluasi kondisi lintas peristiwa dan jendela waktu, bukan hanya baris tunggal.
4. **Alert** — saat sebuah kondisi terpenuhi, membuat sebuah kasus untuk manusia dengan konteks yang cukup untuk bertindak.

## Arsitektur

| Tahap | Komponen | Yang terjadi |
| --- | --- | --- |
| Pengumpulan | Agen atau forwarder (juga tarikan API tanpa agen) | Mengirim log keluar dari setiap host atau layanan; pengiriman off-box yang membuat log tahan manipulasi |
| Parsing dan normalisasi | Parser / pipeline ingest | Mengekstrak timestamp dan field (src_ip, user, event_code) serta memetakannya ke skema bersama |
| Penyimpanan | Tier hot, warm, dan cold | Data terkini disimpan di penyimpanan cepat untuk investigasi; data lebih lama dipindah ke tier yang lebih murah |
| Correlation | Aturan deteksi | Ambang dan urutan yang dievaluasi terhadap data yang sudah dinormalisasi |
| Dashboard | Visualisasi | Kesehatan sumber, tren alert, dan pivot yang digunakan analis selama investigasi |
| Alerting | Routing dan notifikasi | Mengirim peristiwa yang memenuhi syarat ke sistem kasus atau tiket dan membangunkan analis (paging) |

## Ingestion vs Normalization

**Ingestion** adalah memasukkan byte-nya: sebuah agen membaca sebuah file, atau sebuah API mengembalikan rekaman, dan SIEM menerimanya. **Normalization** adalah membuatnya bermakna: baris mentah dipecah menjadi field, timestamp diurai ke format yang umum, dan nama field dipetakan ke skema bersama — misalnya setiap hasil autentikasi menjadi `auth.success=true` atau `auth.success=false`.

Perbedaan ini penting karena ingestion tanpa normalisasi hanya menghasilkan tumpukan teks yang bisa dicari. Anda bisa meng-grep-nya, tetapi tidak bisa menjawab secara andal "kegagalan login per sumber lintas Windows dan Linux" sampai kedua sisi berbicara dengan nama field yang sama.

## Aturan Correlation

Sebuah aturan correlation mengekspresikan kondisi yang hanya berarti jika dikombinasikan: **peristiwa A dalam X menit, ditambah peristiwa B, sama dengan sebuah alert.** Aturan umumnya menentukan lingkup (sumber atau aset mana), ambang (berapa banyak peristiwa), jendela waktu (selama berapa lama), dan severity.

Alert tunggal peristiwa, seperti terlihatnya hash malware yang sudah diketahui berbahaya, sederhana. Aturan sekuens dan ambang menangkap perilaku yang per barisnya tidak berbahaya tetapi mencurigakan secara agregat: kegagalan berulang yang diikuti keberhasilan, beacon yang berulang pada interval tetap, atau akun yang dibuat lalu digunakan dari negara baru.

## Contoh Terinci: Alert Brute Force

Pernyataan aturan: **3 login gagal diikuti 1 login berhasil untuk akun yang sama dari sumber yang sama dalam 5 menit.** Secara individual, login gagal terjadi terus-menerus dan keberhasilan adalah hal normal; bersama-sama keduanya menunjukkan password yang ditebak.

Splunk SPL ilustratif (nama field berbeda antar implementasi, dan jendela waktu pencarian memberikan batas 5 menit):

```spl
index=auth (action=failure OR action=success)
| bin _time span=5m
| stats count(eval(action="failure")) as fails count(eval(action="success")) as ok by _time, src_ip, user
| where fails >= 3 AND ok >= 1
```

Alert yang sampai ke analis harus memuat akun, IP sumber, kedua hitungan, serta waktu peristiwa paling awal dan paling akhir — cukup untuk memutuskan dalam menit pertama apakah perlu dieskalasi.

## Perbandingan Platform

| Platform | Model | Cocok untuk | Catatan |
| --- | --- | --- | --- |
| Splunk | Komersial, berbasis lisensi | Tim besar atau matang yang menghargai bahasa pencarian yang kuat | Mampu dan banyak dipakai; lisensi adalah pendorong biaya utama, verifikasi harga terkini |
| Elastic Stack (ELK) | Tier gratis dikelola sendiri ditambah langganan berbayar | Tim yang ingin menjalankan stack sendiri dan menyimpan data di dalam negeri | Anda yang mengoperasikan klaster: kapasitas, peningkatan versi, dan daya tahan adalah tanggung jawab Anda |
| Wazuh | Gratis dan open source, XDR plus SIEM | Tim dengan anggaran terbatas, lab, dan penengahan berukuran menengah | Memaketkan fitur deteksi, pemantauan integritas, dan incident response |
| Microsoft Sentinel | SaaS cloud-native di ekosistem Microsoft | Organisasi yang sudah berinvestasi pada identitas dan endpoint Microsoft | Layanan terkelola; verifikasi harga terkini, konektor, dan perilaku retensinya |

> Verifikasi sebelum berkomitmen: model harga, edisi, dan fitur cloud sering berubah. Perlakukan setiap angka yang Anda baca sebagai cuplikan dan konfirmasikan di halaman harga dan dokumentasi resmi. Halaman ini sengaja tidak mencantumkan harga pasti apa pun.

## Dasar Bahasa Query

Anda tidak perlu menguasai semua bahasa, tetapi Anda seharusnya mengenali bentuk masing-masing.

### SPL (Splunk)

Berbasis pipe: sebuah pencarian diikuti transformasi.

```spl
index=windows EventCode=4625 | stats count by src_ip | where count > 10
```

### KQL (Kusto Query Language)

Dipakai oleh Microsoft Sentinel dan Defender, bergaya tabel: saring dengan `where`, lalu agregasi dengan `summarize`, misalnya `SecurityEvent | where EventID == 4625 | summarize FailedLogins=count() by IPAddress`.

### Kueri bergaya Lucene

Dipakai oleh sintaks kueri klasik Elastic, dan disederhanakan lagi dalam KQL milik Kibana: pencocokan boolean field-term seperti `event_code:4625 AND NOT user:svc_backup`.

Keterampilan yang bisa dipindahkan adalah bentuk pertanyaannya: saring dulu, lalu agregasi, lalu bandingkan dengan sebuah ambang.

## Apa yang Membuat SIEM Berhasil atau Gagal

1. **Kualitas data.** Sumber yang hilang, kesalahan parsing, dan clock skew membuat setiap aturan tidak andal, sehingga pemantauan kesehatan pengumpulan bukanlah pilihan opsional.
2. **Penyetelan.** Aturan yang terlalu luas membanjiri analis dengan kebisingan; aturan yang terlalu sempit melewatkan aktivitas nyata. Kedua kegagalan ini tampak seperti "SIEM-nya tidak bekerja."
3. **Kekuatan staf.** SIEM tanpa analis dengan jadwal adalah arsip log yang mahal. Alert menciptakan pekerjaan, dan pekerjaan itu membutuhkan pemilik, runbook, serta umpan balik kembali ke para pembuat aturan.

Bangun dalam urutan itu: perbaiki datanya terlebih dahulu, lalu setel apa yang terpicu, lalu skalakan orang dan proses di sekitarnya.

## Lihat juga

- [Sumber Log yang Penting](log-sources.md)
- [Menulis Aturan Sigma](../detection/sigma-rules.md)
- [Penyetelan & False Positive](../detection/false-positives.md)
- [Perangkat Pertahanan](../professional/defense-tooling.md)
