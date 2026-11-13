# Proses Incident Response

Incident response (IR) adalah serangkaian kegiatan yang terstruktur untuk mendeteksi, melakukan containment (pengendalian), eradication (pembersihan), dan recovery (pemulihan) dari sebuah insiden keamanan sembari mempertahankan bukti. Proses yang bersama ini penting karena insiden menimbulkan tekanan waktu, dan keputusan yang terpecah-pecah menyebabkan jendela containment terlewat serta data forensik hilang. Artikel ini mengikuti siklus yang dijelaskan dalam NIST SP 800-61 untuk tim yang membela sistem yang mereka miliki atau berwenang untuk melindungi.

## Apa yang Termasuk Insiden

Insiden keamanan adalah event atau rangkaian event yang benar-benar atau berpotensi melanggar kerahasiaan (confidentiality), integritas, atau ketersediaan (availability) sistem atau data, atau melanggar kebijakan organisasi. Perhatikan kata *berpotensi*: Anda tidak memerlukan bukti eksfiltrasi data untuk menyatakan sebuah insiden — ancaman yang kredibel sudah cukup.

| Kategori | Definisi | Contoh |
|---|---|---|
| Event | Segala kejadian yang dapat diamati; sebagian besar bersifat jinak dan hanya bermakna dalam konteksnya | Seorang pengguna melakukan VPN dari kota baru setelah konferensi |
| Insiden | Sebuah event (atau rangkaian) yang merugikan, atau mengancam untuk merugikan, kerahasiaan, integritas, atau ketersediaan | Sesi tersebut kemudian diikuti oleh pengunduhan massal file pukul 03:00 |
| Near-miss | Sesuatu yang hampir menjadi insiden tetapi diblokir atau dilaporkan sebelum ada kerugian | Pengguna mengeklik tautan phishing tetapi melaporkannya; tidak ada kredensial yang dimasukkan |

Kasus-kasus yang masih ragu sebaiknya dinyatakan sebagai insiden lalu diturunkan kelasnya kemudian jika perlu — biaya investigasi yang tidak perlu jauh lebih rendah daripada kebocoran yang terlewat.

## Siklus Incident Response

Siklus ala NIST SP 800-61 adalah sebuah lingkaran, bukan jalan satu arah: temuan selama deteksi dapat mengembalikan Anda ke fase containment, dan lessons learned (pelajaran yang dipetik) memberi umpan balik pada putaran persiapan berikutnya.

| Fase | Kegiatan inti | Keluaran |
|---|---|---|
| Persiapan (Preparation) | Playbook, peran, tooling, cakupan logging, pelatihan, tabletop exercise | Rencana IR, jadwal piket, SIEM yang berfungsi, backup yang sudah diuji |
| Deteksi & Analisis | Triage alert, penentuan cakupan, klasifikasi severity, pengumpulan evidence, pembuatan timeline | Tiket insiden, pernyataan cakupan, penilaian severity, timeline awal |
| Containment, Eradication & Recovery | Containment jangka pendek, menghilangkan persistence, patch, pemulihan, pemantauan ulang terhadap infeksi berulang | Catatan containment, sistem yang dibersihkan atau dibangun ulang, layanan yang dipulihkan |
| Aktivitas Pasca-Insiden | Meeting lessons learned, metrik, pembaruan deteksi dan playbook | Laporan lessons learned, action item dengan penanggung jawab dan tanggal |

### Deteksi & Analisis dalam Praktik

Triage dimulai dari "apakah ini nyata?" dan segera berlanjut ke "sejauh mana cakupannya?" (lihat [Triage Alert](triage-alerts.md)). Selama penentuan cakupan, tanyakan host, akun, data, dan jendela waktu mana saja yang terdampak. Dua kebiasaan ini mencegah sebagian besar masalah di tahap berikutnya: kumpulkan evidence volatil sebelum menyentuh mesin yang terdampak, dan catat setiap tindakan beserta timestamp saat Anda melakukannya.

Klasifikasi severity (sesuaikan nama dan ambang batas dengan organisasi Anda):

| Severity | Definisi | Contoh | Target respons |
|---|---|---|---|
| Critical | Dampak luas yang sedang berlangsung: eksfiltrasi, ransomware yang menyebar, domain yang jatuh | Enkripsi menyebar ke seluruh file server | Segera |
| High | Kompromi terkonfirmasi dengan potensi pergerakan lateral | Beacon C2 terkonfirmasi di sebuah server | Menit hingga jam |
| Medium | Aktivitas berbahaya terkonfirmasi, cakupan sempit | Satu workstation menjalankan loader; tidak ada penyebaran | Jam hingga sehari |
| Low | Mencurigakan tetapi sempit, dampak pada satu pengguna | Satu blokir malware tanpa eksekusi | Dalam SLA |
| Informational | Jinak atau terblokir penuh tanpa efek | Koneksi keluar ke IP yang diketahui berbahaya, diblokir | Hanya dicatat |

### Strategi Containment

Containment dulu, tetapi pilih trade-off-nya secara sadar — beberapa tindakan menghancurkan evidence atau menyebabkan gangguan yang sebenarnya bisa ditunda:

| Strategi | Jangka pendek (segera) | Tindak lanjut jangka panjang | Trade-off |
|---|---|---|---|
| Isolasi jaringan | Putuskan host atau pindahkan ke VLAN karantina | Blokir indikator C2 di perimeter | Cepat dan sederhana; dapat memberi tahu penyerang dan memutus pemantauan EDR yang sedang berjalan |
| Reset kredensial | Reset akun yang terkompromi, cabut sesi | Rotasi password, tinjauan MFA | Cepat menghentikan penyalahgunaan akun; mengganggu, dapat merusak akun service jika dilakukan tanpa pertimbangan |
| Pemblokiran indikator | Firewall, proxy, atau pemblokiran DNS untuk domain dan IP C2 | Penyesuaian berbasis intelijen | Dampak rendah; penyerang dapat berpindah ke infrastruktur baru |
| Penjagaan evidence | Pertahankan evidence, koordinasikan sebelum remediation | Tinjauan forensik, pengungkapan jika diperlukan | Memaksimalkan evidence; memerlukan persetujuan terdokumentasi |

### Eradication & Recovery

Hilangkan persistence (scheduled task, service, registry run key, langganan WMI, akun liar), patch vektor akses awal, pulihkan data bersih dari backup yang terverifikasi, lalu pantau sistem yang dipulihkan terhadap indikator intrusi yang sama. Recovery dianggap selesai hanya setelah Anda memverifikasi perilaku normal — bangun ulang, jangan sekadar bersihkan, saat kepercayaan Anda rendah.

### Aktivitas Pasca-Insiden

Adakan meeting lessons learned dalam waktu sekitar satu minggu, selagi ingatan masih segar. Pertanyaan yang berguna: Apa vektor akses awalnya? Di mana deteksi kami pertama kali membunyikan peringatan, dan berapa lama waktu yang dibutuhkan? Apa yang kami lakukan sehingga terhambat? Perubahan tunggal apa yang akan menangkap ini lebih cepat kali berikutnya? Jaga rapat tetap blameless — tujuannya adalah memperbaiki celah proses dan deteksi, bukan mencari orang untuk disalahkan; orang berhenti melapor ketika mereka takut disalahkan. Keluarannya adalah laporan tertulis plus action item dengan penanggung jawab dan tanggal yang jelas.

## Dokumentasi Sepanjang Proses

Tulis timeline selama proses berlangsung, bukan dari ingatan setelahnya. Format entri minimum:

```
[YYYY-MM-DD HH:MM TZ] actor: action taken -> result (evidence: host, artifact, ticket)
2026-03-04 14:02 UTC  analyst-jane: isolated WORKSTATION-1148 via EDR -> host offline, ticket IR-412
2026-03-04 14:06 UTC  analyst-jane: collected memory image -> mem_1148.E01, sha256 3f9c... on evidence share
```

Setiap tindakan: siapa, kapan, apa, mengapa, dan di mana evidence disimpan. Timeline ini menjadi masukan bagi laporan insiden, meeting lessons learned, serta proses hukum atau asuransi apa pun yang mengikuti.

## Skenario Mini: Ransomware pada Satu Workstation

| Fase | Tindakan |
|---|---|
| Persiapan (sudah dilakukan) | Playbook ransomware sudah ada; EDR terdeploy; backup sudah diuji; kontak eskalasi diketahui |
| Deteksi | EDR memberi peringatan penghapusan shadow copy dan penamaan massal file di WORKSTATION-1148 |
| Analisis | Triage mengonfirmasi enkripsi sedang berlangsung; pengecekan cakupan menemukan satu host, satu pengguna; severity = High |
| Containment | Isolasi host (pertahankan kanal manajemen EDR jika didukung); reset password pengguna; blokir domain C2 di proxy |
| Eradication | Kumpulkan memory dan disk image; temukan vektor awal (email phishing dengan macro); hilangkan persistence; pindai seluruh armada terhadap indikator yang sama; bangun ulang host dari image yang diketahui baik |
| Recovery | Pulihkan file pengguna dari backup; kembalikan host ke produksi; pantau indikator yang sama selama 7 hari |
| Pasca-insiden | Pelajaran: phishing mencapai inbox, eksekusi macro tidak terblokir, tidak ada jalur pelaporan yang mudah — tindakan: perketat aturan email, terapkan kebijakan macro, promosikan tombol pelaporan |

## Kesalahan Umum

| Kesalahan | Mengapa berdampak buruk | Pendekatan yang lebih baik |
|---|---|---|
| Tidak ada playbook, mengimprovisasi di bawah tekanan | Keputusan tidak konsisten, langkah terlewat | Siapkan playbook per jenis insiden sebelum Anda membutuhkannya |
| Menghapus mesin terlalu cepat | Menghancurkan evidence; akar masalah tidak pernah terungkap | Contain, buat image, lalu bangun ulang |
| Tidak ada timeline selama insiden | Laporan ditulis dari ingatan; perdebatan kemudian | Beri timestamp pada setiap tindakan saat terjadi |
| Menyatakan insiden terlalu telat | Penyerang mendapat waktu untuk menyebar | Nyatakan lebih awal, turunkan kelas nanti |
| Menganggap containment sebagai recovery | Infeksi kembali setelah disambungkan ulang | Verifikasi eradication, pantau setelah recovery |
| Melewatkan meeting lessons learned | Insiden yang sama terulang | Adakan, tulis, tetapkan penanggung jawab |

## Lihat juga

- [Triage Alert](triage-alerts.md)
- [Dasar Digital Forensics](forensics-basics.md)
- [Playbook & Checklist](playbooks.md)
- [Dasar Security Operations](../fundamentals/security-operations.md)
