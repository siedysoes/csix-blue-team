# Playbook & Checklist

Playbook adalah prosedur yang sudah ditulis sebelumnya, langkah demi langkah, untuk menangani satu jenis insiden tertentu, sehingga responder tidak perlu mengimprovisasi saat event sedang berlangsung. Playbook memampatkan pengalaman yang sulit diperoleh menjadi tindakan yang berulang, titik keputusan, dan komunikasi. Playbook melengkapi proses IR yang umum: proses memberi tahu Anda sedang berada di fase mana, sedangkan playbook memberi tahu Anda secara persis apa yang harus dilakukan untuk jenis insiden ini.

## Playbook vs Proses IR

Proses incident response bersifat generik dan berlaku untuk setiap insiden. Playbook bersifat khusus dan tersedia per jenis insiden — ransomware, phishing, kompromi akun, eksfiltrasi data, malware di server. Anda selalu mengikuti prosesnya, dan memilih playbook yang cocok dengan pola yang sedang Anda lihat.

## Anatomi Playbook

| Bagian | Isi |
|---|---|
| Trigger / deteksi | Alert atau kondisi apa yang memulai playbook |
| Langkah triage | Pastikan ini nyata, kumpulkan konteks, klasifikasikan severity |
| Tindakan containment | Langkah berurutan: isolasi, blokir, reset |
| Eskalasi | Siapa yang dihubungi, kapan, dan dengan informasi apa |
| Komunikasi | Template untuk eskalasi internal dan pembaruan kepada pemangku kepentingan |
| Verifikasi pemulihan | Cara memastikan ancaman sudah hilang dan layanan sehat |
| Pelajaran | Apa yang perlu diberi umpan balik ke deteksi dan ke playbook itu sendiri |

Setiap playbook juga memerlukan penanggung jawab yang ditunjuk, versi dan tanggal, serta tautan ke aturan deteksi yang memicunya.

## Contoh Playbook: Login Mencurigakan / Kompromi Akun yang Mungkin

**Trigger:** Alert SIEM untuk impossible travel (perjalanan tidak mungkin), atau login sukses dari negara/ASN yang belum pernah terlihat untuk akun pengguna X.

1. **Konfirmasi dan lakukan enrichment (dalam 5 menit).** Baca detail alert: IP sumber, ASN dan reverse DNS, geolokasi, departemen pengguna, waktu. Periksa apakah IP ini muncul di alert lain.
2. **Tinjau aktivitas akun belakangan ini.** Percobaan gagal sebelum sukses? Perangkat baru terdaftar? Metode MFA ditambahkan atau diubah? Aturan mailbox dibuat, terutama aturan penerusan? Pengunduhan file yang tidak biasa?
   - **Titik keputusan A** — metode MFA atau aturan mailbox diubah selama sesi ini: perlakukan sebagai kompromi terkonfirmasi, lanjut ke langkah 4.
   - **Titik keputusan B** — aktivitas bersih dan pengguna baru bepergian atau memakai VPN: kemungkinan jinak, tetapi tetap verifikasi dengan pengguna melalui kanal terpisah (jangan pernah mengirim email ke akun yang mungkin terkompromi).
3. **Klasifikasikan severity.** Pengguna standar tanpa tindakan mencurigakan: Low/Medium. Akun privilese, atau perubahan persistence apa pun: High/Critical.
4. **Contain.** Cabut sesi dan token menggunakan proses terdokumentasi platform identitas Anda, reset password, nonaktifkan metode MFA yang baru saja ditambahkan, hapus aturan mailbox liar. Isolasi endpoint juga jika malware diduga.
   - **Titik keputusan C** — mencabut sesi untuk akun bersama atau akun service dapat menyebabkan gangguan layanan: periksa ketergantungan terlebih dahulu dan libatkan pemilik layanan sebelum bertindak.
5. **Verifikasi cakupan.** Tanyakan SIEM untuk IP sumber atau ASN yang sama serta akun lain dengan perilaku serupa di jendela waktu yang sama. Ada temuan: buka tiket IR yang lebih luas.
6. **Pulihkan dan kembalikan.** Pulihkan akses pengguna hanya setelah verifikasi, beri tahu pengguna melalui kanal terverifikasi, dan dokumentasikan apa yang berubah dan kapan.
7. **Pasca-insiden.** Tambahkan IP sumber atau ASN ke konteks deteksi, tinjau apakah aturan pemicu perlu penyesuaian, dan catat waktu hingga containment tercapai.

## Membangun Playbook Pertama Anda

1. Pilih satu jenis insiden yang benar-benar Anda jumpai (atau secara kredibel diperkirakan), seperti malware pada sebuah workstation.
2. Tulis apa yang Anda lakukan kali terakhir, secara berurutan, meski berantakan.
3. Pangkas menjadi sepuluh langkah yang wajib terjadi; pindahkan sisanya ke tautan atau lampiran.
4. Tambahkan titik keputusan: di mana seorang analis yang kompeten harus mengambil keputusan?
5. Tambahkan kontak eskalasi dan template komunikasi.
6. Uji melalui tabletop bersama rekan; perbaiki apa pun yang mereka temui hambatan.
7. Tetapkan penanggung jawab dan tanggal tinjauan, lalu tautkan ke aturan deteksi pemicunya.

Playbook pendek yang dipakai mengalahkan playbook sempurna yang tidak pernah dipakai. Jika sebuah langkah memerlukan perintah tool, tautkan ke dokumentasi resmi tool tersebut alih-alih menempelkan perintah yang akan kedaluwarsa.

## Checklist vs Playbook vs Runbook

| Jenis | Cakupan | Digunakan ketika | Contoh |
|---|---|---|---|
| Checklist | Daftar verifikasi singkat | Tugas apa pun, untuk menghindari langkah terlewat | "Sebelum menutup: timeline lengkap, evidence di-hash, pemangku kepentingan diberi tahu" |
| Playbook | Prosedur lengkap untuk satu jenis insiden | Pola insiden yang sudah dikenal telah terpicu | Kompromi akun yang mungkin |
| Runbook | Prosedur operasional untuk sistem atau tugas | Operasi rutin, otomasi, atau langkah pemulihan | "Rotasi kredensial akun service" |

Ketiganya bertingkat: runbook menopang langkah playbook tertentu, dan checklist menjaga kelengkapan playbook.

## Template Komunikasi

Pesan eskalasi internal (sesuaikan dengan lingkungan Anda):

```
[IR-141] Possible account compromise - user j.doe (Finance)
Severity: High | Detected: 2026-03-04 14:02 UTC
Known facts: successful login from hosting-provider IP 198.51.100.10;
forwarding rule created 3 minutes later. No malware found yet.
Scope checked: same IP seen on one other account (k.service).
Actions taken: sessions revoked, password reset, rule removed.
Asks: L2/IR to confirm eradication; identity team to review token revocation.
Contact: analyst-jane in #ir-room, ticket IR-141
```

Struktur pembaruan kepada pemangku kepentingan: (1) status dan severity, (2) hanya fakta yang terkonfirmasi, (3) dampak sejauh ini, (4) tindakan yang sudah diambil, (5) waktu pembaruan berikutnya. Jangan pernah berspekulasi dalam bentuk tulisan — selalu pisahkan fakta terkonfirmasi dari dugaan.

## Menguji Playbook Anda

Tabletop exercise adalah diskusi terfasilitasi di mana tim menelusuri sebuah skenario di atas kertas, tanpa sistem hidup yang terlibat. Untuk menjalankannya secara sederhana: pilih skenario (phishing yang berujung pada penggunaan ulang password), berikan fakta pembuka kepada tim, biarkan mereka berdiskusi tentang langkah-langkah mereka selama 30-60 menit, lalu bahas apa yang gagal. Catat setiap celah sebagai action item. Jaga rapat tetap blameless dan ber-timebox, dan verifikasi bahwa daftar kontak serta akses tool benar-benar berfungsi selama latihan. Ketensi yang umum adalah satu tabletop per kuartal.

## Menjaga Playbook Tetap Hidup

Tinjau playbook secara berkala yang tetap (umumnya setidaknya setahun sekali) **dan perbarui setelah setiap insiden nyata** — setiap insiden memperlihatkan langkah yang salah, hilang, atau terlalu lambat. Ketika aturan deteksi berubah, trigger playbook harus ikut berubah. Berikan setiap playbook sebuah versi dan tanggal "terakhir diverifikasi": playbook yang basi lebih buruk daripada tidak ada, karena menciptakan rasa percaya diri yang palsu. Verifikasi panduan terkini terhadap sumber seperti NIST SP 800-61 dan kebijakan organisasi Anda sendiri.

## Kesalahan Umum

| Kesalahan | Konsekuensi | Perbaikan |
|---|---|---|
| Playbook terlalu panjang (10 halaman) | Tidak ada yang membacanya di tengah insiden | Satu halaman langkah berurutan plus tautan |
| Tidak ada penanggung jawab | Membusuk diam-diam | Tetapkan penanggung jawab dan tanggal tinjauan |
| Tidak pernah diuji | Gagal tepat saat dibutuhkan | Tabletop exercise per kuartal |
| Tidak ada titik keputusan | Tim menebak di bawah tekanan | Cabang if/then yang eksplisit |
| Tidak diperbarui setelah insiden nyata | Hambatan yang sama berulang setiap kali | Pembaruan playbook adalah action item standar |
| Tidak ada kontak eskalasi di dalam playbook | Tertunda saat orang piket tidak terhubung | Verifikasi kontak per kuartal; daftarkan peran, bukan hanya nama |
| Langkah mengasumsikan tool yang tidak diakses siapa pun | Playbook macet di tengah insiden | Periksa akses selama tabletop |

## Lihat juga

- [Proses Incident Response](incident-response.md)
- [Triage Alert](triage-alerts.md)
- [Dasar Digital Forensics](forensics-basics.md)
- [Siklus Deteksi](../detection/detection-lifecycle.md)
