# Karier & Sertifikasi Blue Team

Peran blue team berkisar dari menangani antrian alert di sebuah SOC hingga merancang arsitektur deteksi, dan bidang ini menghargai keterampilan yang dapat dibuktikan sama tingginya dengan kredensial. Artikel ini memetakan tahap karier, peran, dan sertifikasi yang umum agar pemula dapat merencanakan secara realistis. Detail sertifikasi sering berubah — verifikasi detail dan harga terkini di situs resmi masing-masing penyedia.

## Jalur Karier

| Tahap | Peran umum | Keterampilan yang penting pada tahap ini |
|---|---|---|
| 1 | SOC Analyst L1 | Triage alert, membaca log, kebersihan tiket, dasar-dasar jaringan, mengikuti playbook |
| 2 | SOC Analyst L2 | Investigasi lebih dalam, penentuan cakupan, analisis host dan jaringan, kualitas eskalasi, sedikit scripting |
| 3 | L3 / Threat Hunter / Incident Responder | Hunting berbasis hipotesis, forensics, TTP penyerang (mis. MITRE ATT&CK) |
| 4 | Detection Engineer / IR Lead | Detection-as-code (Sigma, YARA), penyesuaian, otomasi, mentoring, pengiriman |
| 5 | Security Architect / Security Manager | Arsitektur, risiko, strategi, anggaran, kepemimpinan lintas tim |

Progresi tidak sepenuhnya linier. Orang berpindah antar peran blue (SOC lalu IR lalu deteksi) dan menyamping dari peran sysadmin, jaringan, atau helpdesk, yang merupakan titik masuk yang umum.

## Contoh Peran

| Peran | Fokus | Kegiatan harian |
|---|---|---|
| SOC Analyst | Mendeteksi dan men-triage alert | Menangani antrian, enrichment, eskalasi atau tutup dengan catatan, melaporkan noise yang jelas |
| Detection Engineer | Membangun dan meningkatkan deteksi | Menulis dan menguji aturan, meninjau celah cakupan, menyesuaikan false positive, mengelola aturan sebagai kode |
| Threat Hunter | Mencari aktivitas tersembunyi | Membentuk hipotesis, menulis kueri lintas log dan endpoint, memvalidasi temuan |
| IR Responder | Contain dan eradication insiden | Memimpin atau mendukung insiden, mengumpulkan evidence, membangun timeline, menjalankan lessons learned |
| DFIR Consultant | Forensics dan IR di lingkungan klien | Penugasan, penanganan evidence, pelaporan; cakupan dan perjalanan bervariasi per firma |

## Perbandingan Sertifikasi

| Sertifikasi | Fokus | Format | Cocok untuk |
|---|---|---|---|
| CompTIA Security+ | Teori keamanan luas dan dasar-dasar | Pilihan ganda dengan item berbasis performa | Tingkat pemula; baseline penyaringan HR yang umum |
| CompTIA CySA+ | Analitik perilaku dan pertahanan | Pilihan ganda, berorientasi analis | SOC analyst di awal karier |
| BTL1 (Blue Team Level 1) | Keterampilan blue hands-on: SIEM, forensics, incident response | Praktis, berbasis lab | Praktisi yang menginginkan latihan terapan |
| GCIA / GCIH (SANS GIAC) | Analisis intrusi / penanganan insiden | Ujian berproktor terikat pelatihan SANS | Analis dengan anggaran pelatihan yang didanai pemberi kerja |
| CISSP | Manajemen keamanan dan pengetahuan domain yang luas | Berproktor; mensyaratkan pengalaman terdokumentasi | Praktisi berpengalaman yang merangsek ke manajemen |

Sertifikasi bukan pengganti keterampilan, tetapi sebagian pemberi kerja dan pasar menggunakannya sebagai saringan. Verifikasi detail dan harga terkini di situs resmi masing-masing penyedia sebelum memutuskan.

## Cara Memilih

- **Mulai dari nol, atau datang dari dukungan IT / jaringan / sysadmin:** mulai dari teori tingkat pemula (gaya Security+) bersamaan dengan lab hands-on gratis.
- **Sudah di SOC atau peran IT dan ingin memperbaiki diri:** pilih opsi yang berorientasi praktisi dan hands-on (BTL1, CySA+, SANS bila didanai pemberi kerja).
- **Mengincar manajemen atau arsitektur dalam lima tahun atau lebih:** kekayaan pengetahuan ala CISSP, setelah Anda memiliki pengalaman operasional nyata.
- Urutan lebih penting daripada merek: satu kredensial pemula plus portofolio nyata mengalahkan daftar panjang sertifikasi lanjutan yang belum Anda raih.

## Keterampilan yang Lebih Penting daripada Kertas

| Keterampilan | Mengapa penting | Cara berlatih |
|---|---|---|
| Nyaman dengan Linux | Sebagian besar tooling dan server berjalan di atasnya | Rusakkan dan perbaiki sebuah VM Linux |
| Scripting (Python, bash) | Mengotomatiskan kueri, parsing, enrichment | Parse log autentikasi, panggil sebuah API, tulis pembantu triage kecil |
| Membaca log dengan lancar | Inti dari pekerjaan deteksi | Baca Windows Event Log dan syslog setiap hari |
| Menulis yang jelas | Laporan dan eskalasi menggerakkan keputusan | Tulis catatan insiden yang bisa diikuti orang asing |
| Dasar-dasar jaringan | Tanpa paket, port, dan DNS, alert tidak masuk akal | Telusuri sebuah login dari klien ke server |

## Membangun Portofolio

- **Home lab:** deploy sebuah SIEM, hasilkan telemetri, tulis dan sesuaikan deteksi (lihat [Lab & Range Pertahanan](defense-labs.md)).
- **Write-up:** terbitkan write-up lab atau CTF yang menjelaskan apa yang Anda lakukan dan mengapa — menulis yang jelas sendiri sudah merupakan bukti keterampilan.
- **Open source:** berkontribusi pada aturan deteksi, dokumentasi, atau perbaikan untuk proyek blue team. Kontribusi kecil yang nyata mengalahkan klaim.
- **Pertahankan daftar pencapaian:** insiden yang ditangani, aturan yang dirilis, perbaikan time-to-detect, beserta tanggalnya.

## Rencana Tahun Pertama

| Bulan | Fokus |
|---|---|
| 1-3 | Fondasi: jaringan, dasar OS, Linux, satu jalur sertifikasi pemula, membaca log setiap hari |
| 4-6 | Lab: SIEM plus beberapa endpoint, deteksi dasar, mulai scripting dengan Python atau bash |
| 7-9 | Keterampilan praktisi: latihan triage di range gratis, sertifikasi praktis pertama atau proyek setara |
| 10-12 | Portofolio: terbitkan dua atau tiga write-up, berkontribusi pada satu aturan deteksi atau dokumentasi tool, melamar peran L1/L2 dengan bukti konkret |

Sesuaikan kecepatan dengan latar belakang Anda, tetapi jaga lab tetap berjalan sepanjang waktu — pemberi kerja menanyakan apa yang telah Anda bangun dan apa yang telah Anda pertahankan.

## Lihat juga

- [Tool Pertahanan](defense-tooling.md)
- [Lab & Range Pertahanan](defense-labs.md)
- [Apa Itu Blue Team](../fundamentals/what-is-blue-team.md)
- [Proses Incident Response](../response/incident-response.md)
