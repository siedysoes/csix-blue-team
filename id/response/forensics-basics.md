# Dasar Digital Forensics

Digital forensics adalah pengumpulan, pelestarian, dan analisis bukti digital secara sistematis untuk merekonstruksi apa yang terjadi dan mengapa. Forensics memperluas triage dengan menghasilkan fakta yang dapat dipertahankan — sebuah timeline, akar masalah, dan bukti cakupan — yang tidak dapat diberikan oleh satu alert saja. Artikel ini membahas prinsip-prinsip, sumber artifact, dan tooling dasar untuk memeriksa sistem yang Anda miliki atau berwenang untuk lindungi.

## Apa yang Ditambahkan Forensics di Luar Triage

Triage menjawab "apakah ini buruk saat ini?" Forensics menjawab: Apa yang sebenarnya terjadi? Kapan mulainya? Apa yang diakses? Bagaimana penyerang masuk? Jawaban-jawaban itu diperlukan untuk timeline yang akurat, untuk pelaporan hukum atau regulasi, untuk lessons learned, dan untuk memastikan eradication benar-benar berhasil. Jika triage adalah sebuah cuplikan, forensics adalah keseluruhan ceritanya.

## Prinsip Penanganan Evidence

| Prinsip | Artinya | Praktik |
|---|---|---|
| Integritas | Evidence harus terbukti tidak berubah | Hash image (mis. SHA-256) sebelum dan sesudah transfer; analisis salinan, jangan pernah aslinya |
| Chain of custody | Dokumentasikan siapa yang menangani evidence dan kapan | Log evidence: item, hash, pengumpul, waktu, lokasi penyimpanan, setiap perpindahan |
| Order of volatility | Kumpulkan data yang paling cepat menghilang terlebih dahulu | Memory, lalu koneksi jaringan, proses yang berjalan, disk, kemudian log yang diarsipkan |
| Interaksi minimum | Setiap tindakan pada sistem mengubah keadaannya | Minimalkan tindakan pada host yang hidup; utamakan image forensik; catat apa yang Anda lakukan |

**Catatan live acquisition:** sebagian evidence hanya ada selagi mesin berjalan — isi RAM, volume terenkripsi yang terpasang, koneksi jaringan aktif. Live acquisition dibenarkan ketika sistem harus tetap hidup atau enkripsi seluruh disk akan membuat image mati tidak terbaca, tetapi cara ini mengubah keadaan, jadi catat setiap perintah yang Anda jalankan beserta outputnya.

## Di Mana Evidence Berada

| Lokasi | Artifact | Yang dapat ditunjukkannya |
|---|---|---|
| Disk - file system | MFT ($MFT), $UsnJrnl, $LogFile | Waktu dan jalur file saat dibuat/diubah/diakses/dihapus, bahkan setelah penghapusan |
| Disk - Windows registry | Run key, service, ShimCache/AmCache, UserAssist, riwayat USB | Persistence, program yang dieksekusi, perangkat yang dipasang |
| Disk - prefetch | File prefetch di `C:\Windows\Prefetch` | executable mana yang berjalan dan kapan waktunya (di mana fitur diaktifkan) |
| Disk - shellbags | Kunci BagMRU | Folder yang dijelajahi di Explorer, termasuk yang terhapus dan jalur jaringan |
| Disk - artifact browser | History, cookie, unduhan, cache | Aktivitas web, C2 melalui browser, kunjungan phishing kredensial |
| Memory | Proses, koneksi jaringan, modul yang dimuat, kode yang disuntikkan | Malware tanpa file, koneksi C2 yang hidup, injeksi kode |
| Log - sentral | Pengumpul SIEM atau syslog | Korelasi lintas host; bertahan terhadap manipulasi endpoint |
| Log - lokal | Windows Event Log, syslog/journald Linux | Detail tingkat host; dapat dihapus oleh penyerang |
| Cloud | Jejak audit penyedia (log audit control-plane, log masuk SaaS) | Siapa mengubah apa, dari mana, kapan — sering kali satu-satunya catatan |

Semantik timestamp (dibuat vs diubah, UTC vs waktu lokal) dan lokasi artifact bervariasi menurut sistem operasi dan versi — verifikasi terhadap referensi terkini untuk lingkungan Anda.

## Alur Kerja Pengumpulan yang Praktis

1. Konfirmasi otorisasi dan cakupan: sistem mana, jendela waktu mana, persetujuan siapa.
2. Tangkap data volatil terlebih dahulu: memory, proses yang berjalan, koneksi jaringan, pengguna yang masuk.
3. Tentukan live atau dead acquisition (lihat catatan live acquisition di atas) dan catat pilihan tersebut.
4. Buat image forensik atau pengumpulan bertarget, lalu hitung hash segera.
5. Catat chain of custody: siapa, kapan, dari mana, disimpan di mana, nilai hash.
6. Analisis hanya salinan; jaga image asli tetap write-protected.
7. Verifikasi ulang hash setelah analisis dan pastikan masih cocok.

## Analisis Timeline

Analisis timeline berarti mengorelasikan timestamp artifact menjadi satu cerita yang utuh: tindakan pengguna, lalu eksekusi proses, lalu file ditulis, lalu persistence dibuat, lalu koneksi jaringan. Pekerjaan praktisnya adalah normalisasi — timestamp berasal dari sumber yang berbeda dengan jam, zona waktu, dan makna yang berbeda. Konversikan semuanya ke satu zona waktu (biasanya UTC), catat clock skew antar host, dan selalu pasangkan setiap timestamp dengan artifact-nya serta tafsiran Anda terhadapnya.

Aturan praktis saat mengorelasikan:

- Satu artifact membuktikan satu fakta; rantai artifact yang independen membuktikan sebuah narasi.
- Utamakan artifact yang kemungkinan besar tidak dimanipulasi penyerang (log sentral hidup lebih lama daripada log lokal).
- Catat kapan timestamp sebuah artifact adalah waktu penulisan terakhir, bukan waktu kejadian.
- Tandai secara eksplisit celah dalam cakupan ("tidak ada data EDR sebelum agen dipasang") alih-alih mengisinya secara diam-diam.

## Jebakan Umum

| Jebakan | Konsekuensi | Kebiasaan yang lebih baik |
|---|---|---|
| Menganalisis sistem yang hidup, bukan image | Timestamp berubah, evidence hancur | Image dulu, analisis salinannya |
| Hanya mengumpulkan artifact yang memicu alert | Cakupan dan akar masalah terlewat | Kumpulkan sesuai order of volatility, baru persempit |
| Mengabaikan zona waktu dan clock skew | Urutan kejadian dalam cerita menjadi salah | Normalisasi ke UTC, catat selisihnya |
| File evidence tanpa label | Tidak bisa menentukan host mana atau kapan | Konvensi penamaan plus log evidence sejak awal |
| Menghapus data "tidak relevan" untuk menghemat ruang | Konteks hilang saat peninjauan | Kumpulkan lebih banyak, saring saat analisis |

## Tooling

Gambaran singkat; semuanya memiliki dokumentasi resmi, dan versi sering berubah:

- **Autopsy** — platform digital forensics open source untuk disk image: analisis file system, tampilan timeline, pencarian kata kunci.
- **Plaso / log2timeline** — mengurai banyak sumber artifact menjadi satu "super-timeline" untuk korelasi berskala besar.
- **KAPE** — pengumpulan artifact bertarget: ambil hanya artifact yang Anda butuhkan, dengan cepat, dari satu atau banyak endpoint (berfokus pada Windows).
- **Volatility** — kerangka kerja analisis memory: analisis proses, jaringan, dan injeksi dari sebuah memory dump.

Verifikasi versi terkini, ketentuan lisensi, dan dokumen penggunaan di situs resmi masing-masing proyek sebelum mengandalkannya dalam investigasi.

## Contoh Mini-Timeline

Kasus: workstation WS-1148, diduga phishing.

| Waktu (UTC) | Artifact | Tafsiran |
|---|---|---|
| 2026-03-04 13:47 | History browser: `invoice_2026.zip` diunduh | Pengguna mengambil lampiran dari situs eksternal |
| 2026-03-04 13:49 | MFT: `invoice_2026.exe` dibuat di Downloads | Arsip diekstrak; executable ada |
| 2026-03-04 13:49 | Prefetch: `invoice_2026.exe` dieksekusi | Binary tersebut berjalan di host ini |
| 2026-03-04 13:50 | Registry Run key: entri `updater` baru | Persistence terbentuk |
| 2026-03-04 13:51 | Memory: proses anak yang mencurigakan dengan koneksi ke 203.0.113.44 | Kemungkinan C2 — eskalasi dan contain sekarang |

Lima observasi yang tidak berhubungan menjadi narasi yang dapat dipertahankan dengan titik awal cakupan yang jelas: segala sesuatu setelah 13:47 berada dalam cakupan.

## Catatan Hukum dan Kebijakan

Materi ini bersifat edukatif. Pemeriksaan forensik dan penanganan evidence harus mengikuti kebijakan organisasi Anda serta hukum lokal yang berlaku, termasuk aturan mengenai persetujuan, perlindungan data, privasi karyawan, dan retensi. Jika ragu, libatkan tim hukum atau kepatuhan Anda sebelum mengumpulkan atau menganalisis data.

## Lihat juga

- [Proses Incident Response](incident-response.md)
- [Triage Alert](triage-alerts.md)
- [Sumber Log yang Penting](../fundamentals/log-sources.md)
- [Playbook & Checklist](playbooks.md)
