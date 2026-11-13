# Sumber Log yang Penting

Log adalah bahan baku pertahanan: tanpanya tidak ada deteksi, tidak ada investigasi, dan tidak ada cara untuk membuktikan apa yang terjadi. Keterampilan praktisnya adalah mengetahui sumber mana yang mengungkap paling banyak, dalam urutan mana harus mengumpulkannya, dan apa yang tidak pernah bisa Anda lihat. Halaman ini adalah peta sumber log yang diprioritaskan untuk lingkungan kecil dan menengah.

## Mengapa Log Penting

Setiap aturan deteksi, hunting, dan timeline forensik dibangun dari peristiwa yang seseorang pilih untuk mencatatnya. SIEM dengan input yang buruk menghasilkan jawaban yang meyakinkan tetapi salah; lingkungan sederhana dengan lima sumber yang tepat bisa mengungguli lingkungan mahal yang punya celah. Karena itu, pemilihan sumber adalah keputusan berdampak tertinggi dalam sebuah program deteksi.

## Sumber Log Prioritas

| Sumber log | Peristiwa kunci | Informasi yang diberikan |
| --- | --- | --- |
| Windows Security event log | 4624 (logon berhasil), 4625 (logon gagal), 4688 (pembuatan proses), 4720 (akun pengguna dibuat) | Siapa yang login dan dari mana, upaya mana yang gagal, proses apa yang berjalan, kapan akun dibuat |
| Sysmon (Windows) | 1 (proses dibuat dengan hash dan command line), 3 (koneksi jaringan), 11 (file dibuat) | Detail endpoint yang kaya di luar logging bawaan Windows: command line, koneksi keluar, penaruhan file (file drop) |
| Log auth dan audit Linux | /var/log/auth.log atau secure (sshd, sudo), aturan auditd | Upaya dan kegagalan login, eskalasi hak akses (privilege escalation), akses perintah dan file di host Linux |
| Firewall dan proxy | Sesi yang diizinkan dan ditolak, permintaan proxy serta keputusannya | Lalu lintas perimeter, layanan yang terekspos, tujuan keluar, aktivitas web yang diblokir dan diizinkan |
| Log server DNS | Kueri dan respons | Domain mana yang diresolusi host Anda, sinyal awal command-and-control dan tunneling |
| Telemetri EDR | Pohon proses, eksekusi skrip, perilaku tanpa file (fileless), status isolasi | Perilaku endpoint dengan konteks, ditambah kemampuan mengisolasi host seketika |
| Log audit cloud (AWS CloudTrail, Azure Activity, GCP Audit Logs) | Panggilan API, login console, perubahan IAM dan kunci | Siapa yang mengubah sumber daya cloud, membuat kredensial, atau membuka penyimpanan ke internet |
| Log akses web server | Permintaan, kode status, IP sumber | Pemindaian (scanning), upaya eksploitasi, dan pola permintaan yang tidak biasa terhadap aplikasi Anda |
| Log gateway email | Pesan yang dikarantina, diterima, dan diblokir; keputusan URL dan lampiran | Pengiriman phishing, upaya impersonasi, dan apa yang sebenarnya diterima pengguna |

### Dua catatan tentang event Windows

- **4624 dan 4625 adalah tulang punggung deteksi autentikasi:** lonjakan 4625 yang berulang menandakan brute force, sedangkan 4624 dengan tipe logon yang tidak lazim menandakan lateral movement.
- **Auditing command line 4688 dan Sysmon harus diaktifkan secara eksplisit.** Instalasi default tidak memberi Anda detail yang diasumsikan sebagian besar deteksi.

## Set Awal untuk Organisasi Kecil

Jika Anda membangun pengumpulan dari nol, susun pekerjaannya seperti ini:

1. **Identitas** — log Windows Security pada domain controller (4624, 4625, 4720 dan peristiwa perubahan akun terkait), atau log login dari penyedia identitas cloud Anda.
2. **Endpoint** — EDR di setiap server dan workstation yang bisa Anda jangkau; di mana EDR tidak mungkin, setidaknya kumpulkan pembuatan proses (4688 atau Sysmon event 1).
3. **Perimeter** — log firewall, termasuk koneksi yang ditolak.
4. **DNS** — log kueri resolver internal Anda, kirim keluar dari host seketika.
5. **SaaS dan cloud** — log audit Microsoft 365 atau Google Workspace, ditambah AWS CloudTrail, Azure Activity, atau GCP Audit Logs jika berlaku.

> **Lima sumber pertama:** log autentikasi domain controller, telemetri EDR, log firewall, log kueri DNS, serta log audit SaaS atau cloud Anda. Bersama-sama mereka menutupi penyalahgunaan identitas, eksekusi endpoint, batas jaringan, callback command-and-control, dan perubahan control plane cloud — yang merupakan hal pertama yang biasanya diminta oleh sebagian besar investigasi.

## Seperti Apa Satu Peristiwa

Normalization (normalisasi) mengubah baris-baris seperti ini menjadi field terstruktur yang bisa Anda agregasi lintas sumber:

```
Windows 4625 (raw, simplified):
  Time=2026-11-12T09:14:02Z  Host=dc01  ID=4625  LogonType=3
  Account=j.mayer  SrcIP=10.20.4.55  Status=0xC000006D

Linux auth.log (raw):
  Nov 12 09:14:07 app01 sshd[4412]: Failed password for j.mayer
  from 10.20.4.55 port 51224 ssh2

Normalized into the same schema:
  event=auth.failure  user=j.mayer  src_ip=10.20.4.55  outcome=failure
```

Setelah kedua sisi berbagi nama field, sebuah aturan bisa berbunyi "lima kegagalan dari satu sumber dalam lima menit" dan berarti hal yang sama untuk login Windows, Linux, VPN, dan cloud.

## Volume vs Nilai

Sumber-sumber berbeda sangat jauh dalam ukuran, dan besar tidak berarti lebih baik:

- **Volume tinggi, nilai tinggi:** pembuatan proses dan DNS. Kebisingan bisa dikelola setelah field dinormalisasi dan aturan disetel; menjatuhkan sumber-sumber ini untuk menghemat penyimpanan biasanya lebih mahal dalam bentuk deteksi yang terlewat daripada yang dihemat.
- **Volume tinggi, nilai sedang:** log proxy dan akses web lengkap. Simpan agar bisa dicari untuk minggu-minggu terkini, arsipkan sisanya, dan saring tujuan yang sudah diketahui benign di sumbernya bila memungkinkan.
- **Volume rendah, nilai kritis:** pembuatan akun, perubahan IAM, penolakan firewall ke antarmuka manajemen. Ini murah untuk disimpan dan tidak boleh pernah di-sampling.

Keputusan penyimpanan seharusnya mengikuti kebutuhan investigasi, bukan hanya anggaran: tanyakan sumber mana yang benar-benar dibutuhkan insiden terakhir Anda, dan simpan sumber-sumber tersebut dalam status panas (hot).

## Dasar Retensi

Sebagai panduan umum, simpan sumber yang Anda gunakan untuk menyelidiki dalam status **bisa dicari setidaknya 90 hari.** Investigasi secara rutin menjangkau berminggu-minggu ke belakang, melalui keterlambatan pelaporan pengguna dan dwell time penyerang, dan banyak tim menemukan bahwa 90 hari adalah titik di mana pivot rutin berhenti gagal. Arsipkan data yang lebih lama secara murah di object storage atau tier terkompresi saat regulasi atau kebijakan mensyaratkan periode yang lebih panjang.

Retensi bergantung pada konteks. Konfirmasikan kebutuhan terhadap kewajiban hukum, regulasi, dan privasi untuk industri serta wilayah Anda, alih-alih memperlakukan 90 hari sebagai aturan universal.

## Pastikan Pengumpulan Benar-Benar Bekerja

Menambahkan sumber ke sebuah konfigurasi bukan berarti Anda menerima peristiwa dari sumber tersebut. Validasi setiap sumber baru dengan checklist singkat:

- Buat sendiri peristiwa yang sudah diketahui, misalnya login gagal, dan temukan di SIEM dalam beberapa menit.
- Bandingkan hitungan: kira-kira berapa peristiwa per jam yang dihasilkan sumber ini, dan apakah itu yang benar-benar masuk?
- Periksa bahwa timestamp jatuh di zona waktu yang benar dan field berhasil diurai ke kolom yang tepat.
- Verifikasi sumber tetap mengirim saat buffer-nya penuh atau koneksi jaringannya putus sebentar.
- Buat alert untuk keheningan: sumber yang berhenti mengirim harus membangunkan seseorang, karena keheningan tampak sehat.

Sumber yang tidak diverifikasi siapa pun cenderung gagal secara diam-diam, dan celah yang diam-diam hanya ditemukan saat terjadi insiden.

## Celah yang Umum

- **Endpoint tanpa EDR** — sistem warisan, perangkat yang tidak dikelola, atau server yang terlupakan justru menjadi tak terlihat di tempat yang paling disukai penyerang.
- **DNS tanpa log** — klien yang memakai resolver publik, atau resolver internal yang log kuerinya tidak pernah dikirim siapa pun, menghapus sinyal callback paling awal Anda.
- **Detail tidak diaktifkan** — auditing pembuatan proses mati, Sysmon tidak pernah dipasang, auditing command line dinonaktifkan: sumbernya ada tetapi kosong.
- **Control plane cloud dibiarkan lokal** — CloudTrail atau log Activity yang tidak diekspor siapa pun bisa hilang bersama akunnya atau karena dihapus penyerang.
- **Clock skew** — host dengan timestamp yang menyimpang membuat correlation menjadi tidak andal; sinkronkan semuanya ke sumber waktu yang konsisten.
- **Log hanya ada di host** — penyerang dengan hak administratif menghapus log lokal, jadi kirim log ke luar (off-box) secara near real time.

## Verifikasi Sebelum Membangun

Event ID, subkategori audit, dan layanan logging cloud berubah seiring waktu, dan sumber-sumber baru bermunculan secara teratur. Periksa dokumentasi resmi Microsoft, AWS, Azure, dan GCP, serta referensi dari vendor perangkat Anda, sebelum memfinalisasi parser, dashboard, atau kebijakan retensi yang dibangun di atas tabel ini.

## Lihat juga

- [Dasar SIEM](siem-basics.md)
- [Menulis Aturan Sigma](../detection/sigma-rules.md)
- [Perangkat Pertahanan](../professional/defense-tooling.md)
- [Threat Hunting](../detection/threat-hunting.md)
