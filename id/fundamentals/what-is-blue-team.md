# Apa Itu Blue Team

Blue team adalah sisi pertahanan dari keamanan informasi: orang-orang yang memantau sistem, mendeteksi serangan, merespons insiden, dan meningkatkan perlindungan di sekitar infrastruktur yang mereka miliki atau berwenang untuk lindungi. Memahami bagaimana blue team, red team, dan purple team saling berkaitan adalah titik awal bagi siapa pun yang mempertimbangkan karier di bidang keamanan pertahanan. Halaman ini memetakan warna tim, struktur organisasi keamanan modern, dan pola pikir yang dipegang para defender.

## Blue, Red, dan Purple

| Tim | Peran | Kegiatan umum |
| --- | --- | --- |
| Blue team | Bertahan, mendeteksi, merespons | Triage alert, monitoring, penyetelan deteksi (detection tuning), incident response, hardening, manajemen log |
| Red team | Menemukan kelemahan dengan menyerang secara aman | Penetration testing, adversary emulation, latihan purple team, laporan temuan dan uji ulang |
| Purple team | Berkolaborasi untuk memvalidasi dan meningkatkan | Menjalankan teknik bersama para defender, memastikan deteksi benar-benar terpicu, menutup celah saat itu juga |

Warna-warna tersebut menggambarkan fungsi, bukan jabatan. Perusahaan kecil mungkin punya satu orang yang melakukan ketiganya; perusahaan besar mungkin punya departemen terpisah untuk masing-masing.

### Tanggung jawab blue team dalam praktik

- Mengawasi antrean alert dan menyelidiki aktivitas mencurigakan.
- Menjaga aliran sumber log tetap lancar dan aturan deteksi tetap akurat.
- Memimpin atau mendukung incident response saat sesuatu terkonfirmasi.
- Mengeras sistem (hardening), menerapkan patch, dan menutup celah konfigurasi setelah insiden.
- Mengukur dan melaporkan kinerja deteksi serta respons.

### Tanggung jawab red team dalam praktik

- Memalsukan (mereplikasi) teknik penyerang yang realistis terhadap sistem yang berwenang mereka uji.
- Tetap dalam lingkup yang disepakati, aturan keterlibatan (rules of engagement), dan jendela waktu yang ditentukan.
- Menghasilkan bukti yang bisa ditindaklanjuti blue team: jalur serangan, artefak, serta deteksi yang tidak pernah terpicu.

## Bagaimana Tim Keamanan Modern Disusun

Sebagian besar organisasi keamanan membagi pekerjaan ke dalam fungsi-fungsi yang terspesialisasi, bahkan ketika satu orang menangani beberapa peran:

- **SOC (Security Operations Center)** — terus-menerus mengawasi telemetri dan alert untuk mendeteksi ancaman tepat saat terjadi.
- **Incident Response (IR)** — mengandung, memberantas, dan memulihkan dari insiden yang terkonfirmasi, lalu menjalankan tinjauan pasca-insiden.
- **Detection Engineering** — membangun, menguji, dan menyetel aturan serta pipeline telemetri yang mengubah peristiwa mentah menjadi alert yang tepercaya.
- **Threat Intelligence** — melacak adversary, kampanye, dan indikator agar pertahanan bersiap menghadapi apa yang realistis mungkin terjadi.
- **GRC (Governance, Risk, and Compliance)** — memetakan kebutuhan bisnis dan regulasi ke kebijakan, kontrol, dan audit.

### Contoh jalur karier

Progresi yang umum adalah: analis SOC L1 (triage) ke analis L2 (investigasi) ke pemburu (hunter) L3 atau detection engineer, lalu incident responder, team lead, atau security engineer. Struktur ini memberi pemula titik masuk yang jelas dan memberi ruang bagi orang berpengalaman untuk mendalami spesialisasi.

## Blue dan Red Saling Melengkapi

Red team ada untuk membuat blue team lebih baik, bukan untuk mengalahkannya. Keluaran yang berguna dari uji yang sah bukanlah skor; melainkan daftar hal yang tidak bisa dilihat pertahanan atau tidak bisa dihentikan tepat waktu. Blue team diuntungkan karena red team mengungkap titik buta: sumber log yang tidak pernah diaktifkan siapa pun, aturan yang tidak pernah terpicu, atau langkah respons yang terlalu lama.

Hubungannya berjalan dua arah. Blue team memberi red team umpan balik jujur ("kami tidak pernah melihat itu") dan membagikan seperti apa alat pertahanan yang sebenarnya dipakai, sehingga latihan berikutnya menguji hal yang penting, bukan hal yang mudah.

## Purple Teaming sebagai Kolaborasi

Purple teaming adalah mode kerja, bukan kotak perkakas terpisah: penyerang dan defender menjalankan teknik berdampingan, mengamati apa yang dihasilkan pertahanan, dan memperbaiki celah selama sesi masih berlangsung.

| Aktivitas purple | Contoh | Hasil bagi defender |
| --- | --- | --- |
| Uji Atomic Red Team | Menjalankan satu teknik kecil yang sudah dipetakan, misalnya membuat scheduled task atau proses parent-child yang mencurigakan | Memastikan sumber log yang diperlukan benar-benar dikumpulkan dan aturan deteksi yang cocok terpicu |
| Latihan assumed breach | Berangkat dari workstation yang sudah terkompromi dan bergerak menuju target berharga | Memvalidasi deteksi lateral movement dan mengukur waktu kontainment |
| Tinjauan celah deteksi | Membandingkan cakupan teknik ATT&CK terhadap aturan dan data yang ada | Backlog terprioritas untuk deteksi baru atau yang diperbaiki |
| Ronde remediasi langsung | Defender memperbaiki sebuah celah, lalu penyerang mengulang teknik yang sama | Uji ulang langsung dengan hasil lulus atau gagal yang terlihat jelas |

Atomic Red Team dan kerangka kerja sejenis menyediakan uji kecil yang berulang dan dipetakan ke teknik ATT&CK, sehingga sesi tetap terfokus dan tidak berubah menjadi engagement penuh.

## Mengapa Pertahanan adalah Jalur Karier yang Kuat

- **Permintaan yang stabil.** Setiap organisasi yang menjalankan sistem membutuhkan orang untuk mengawasi, mencatat, dan membela sistem tersebut, dan jumlah sistem tumbuh lebih cepat daripada jumlah defender.
- **Struktur yang jelas.** Peran SOC bertingkat, metrik yang terdefinisi, dan proses yang terdokumentasi memungkinkan seseorang memulai dengan pengalaman terbatas dan berkembang secara terencana.
- **Peningkatan yang terukur.** Kemajuan terlihat: waktu deteksi lebih singkat, false positive lebih sedikit, cakupan lebih baik setiap siklus penyetelan.
- **Hambatan masuk yang rendah.** Home lab, perangkat open-source, dan repositori deteksi publik memungkinkan siapa saja berlatih tanpa membeli perangkat lunak penyerangan atau membutuhkan izin khusus.

### Titik masuk yang realistis

- Analis SOC L1, peran pertama yang paling umum dan yang disiapkan repositori ini untuk Anda.
- Administrator IT, operator jaringan, atau engineer helpdesk yang pindah ke posisi keamanan dengan pengetahuan domain yang tetap terjaga.
- Lulusan atau rotasi magang di dalam tim keamanan.
- Portofolio publik: tulisan lab, aturan deteksi, dan keputusan penyetelan yang terdokumentasi yang bisa dibaca pewawancara.
- Sertifikasi yang dipakai sebagai sinyal penyaringan dan kerangka belajar, bukan sebagai pengganti latihan praktis.

## Pola Pikir Defender

Dua kebiasaan mendefinisikan defender yang baik:

1. **Asumsikan terjadinya breach (assume breach).** Perimeter bisa gagal dan pengguna bisa membuat kesalahan. Alih-alih bertaruh pada pencegahan saja, rencanakan untuk penyerang yang sudah berada di dalam: apa yang akan mereka sentuh, dan log mana yang akan menampilkannya?
2. **Deteksi yang cepat.** Anda tidak akan menghentikan semuanya. Tujuannya adalah memperpendek jendela antara tindakan penyerang dan kesadaran Anda akan hal itu, lalu merespons ketika kerusakan masih terbatas.

Kebiasaan pendukung: utamakan kedalaman daripada kesempurnaan (banyak kontrol kecil daripada satu solusi megah), perlakukan setiap insiden dan setiap false positive sebagai umpan balik, dan dokumentasikan keputusan agar analis berikutnya tidak mulai dari nol.

## Sehari dalam Kehidupan

**07:50** — Serah terima dari shift malam: dua tiket terbuka, satu host diisolasi menunggu analisis, selain itu sepi. **08:15** — Triage antrean: 30 alert semalaman, sebagian besar dari pemasangan software yang sudah dikenal, ditutup secara massal dengan catatan referensi. **09:40** — Satu alert menonjol: PowerShell terenkripsi yang dijalankan Word di sebuah workstation keuangan. Perkaya di EDR, tinjau pohon proses (process tree), temukan koneksi keluar ke sebuah domain yang jarang. Eskalasi ke L2, isolasi host, tambahkan domain ke blocklist, dan berburu pola yang sama di seluruh armada host. **11:00** — Dokumen berbahaya terkonfirmasi; playbook respons dijalankan dan indikator dibagikan secara internal. **14:00** — Sesi detection engineering: aturan yang terpicu sudah benar tetapi berisik, sehingga ambangnya disetel dan aturan yang diperbarui masuk ke repositori tim. **16:30** — Perbarui tiket, tulis catatan shift, serahkan item yang masih terbuka. Hari itu sebagian besar rutin, dan justru itulah mengapa satu alert yang tidak biasa itu penting.

## Ruang Lingkup Repositori Ini

Repositori ini mencakup sisi **pertahanan** dari keamanan untuk sistem yang Anda miliki atau berwenang Anda lindungi: operasi, deteksi, respons, serta alat di baliknya. Materi ofensif, termasuk metodologi penetration testing dan analisis malware untuk pemahaman pertahanan, terdapat di sumber daya pendamping:

- Pembelajaran pentest: https://siedysoes.github.io/csix-pentest/
- Pembelajaran lab malware: https://siedysoes.github.io/csix-malware-lab/

Semua isi di sini bersifat edukatif dan berangkat dari sudut pandang pertahanan. Berlatihlah hanya pada sistem yang Anda miliki atau yang secara eksplisit Anda izinkan untuk diuji.

## Lihat juga

- [Dasar Security Operations](security-operations.md)
- [Siklus Hidup Deteksi](../detection/detection-lifecycle.md)
- [Proses Incident Response](../response/incident-response.md)
- [Karier & Sertifikasi Blue Team](../professional/blue-team-career.md)
