# Lab & Range Pertahanan

Pembela membutuhkan lab dengan alasan yang sama seperti penyerang berlatih: Anda belajar deteksi dan respons dengan merusak dan mengamati sistem yang Anda kendalikan. Sebuah lab SOC rumahan memungkinkan Anda menghasilkan telemetri nyata, menguji deteksi, dan berlatih incident response tanpa menimbulkan risiko pada produksi. Artikel ini memberikan arsitektur, langkah pembangunan, dan rencana 30 hari untuk sebuah lab yang terisolasi.

## Mengapa Pembela Membutuhkan Lab

- Anda dapat memicu alert Anda sendiri dan menyeluruh jalurnya: tindakan, log, aturan, alert, catatan triage.
- Kegagalan yang aman: rusakkan deteksi dan aturan tanpa dampak bisnis.
- Bukti konkret untuk wawancara dan portofolio: lab yang Anda bangun mengalahkan klaim yang samar.
- Berlatih respons: isolasi, kumpulkan evidence, tulis timeline, jalankan satu putaran lessons learned.

## Arsitektur Lab SOC Rumahan

```text
        [ Host-only network: 192.168.56.0/24  (NAT for updates only) ]

 +--------------------+    +--------------------+    +--------------------+
 | Windows Domain     |    | Windows            |    | Linux Server       |
 | Controller         |<-->| Workstation        |    | (web app target)   |
 | (AD, DNS, GPO)     |    | (user, Office)     |    |                    |
 +--------------------+    +--------------------+    +--------------------+
          |                        |                         |
          +-----------+------------+-------------------------+
                      |
              +--------------------+       +------------------------+
              | Wazuh Manager      |<------| Attacker VM            |
              | (SIEM/XDR, agents, | logs  | (Kali or atomic test   |
              |  rules, dashboards)|       |  runner)               |
              +--------------------+       +------------------------+

   [ Analysis host ] -- runs Autopsy/Velociraptor and the SIEM dashboards
```

Letakkan VM penyerang di jaringan terisolasi yang sama agar trafiknya tidak pernah keluar dari mesin Anda.

## Langkah Pembangunan

| Komponen | Tujuan | Catatan |
|---|---|---|
| Hypervisor (VirtualBox, VMware, atau Hyper-V) | Menampung semua VM | Ambil snapshot sebelum setiap latihan: itu tombol undo Anda |
| Windows Domain Controller | AD, DNS, GPO untuk telemetri identitas yang realistis | Beri baseline dengan beberapa akun pengguna |
| Windows workstation | Menghasilkan event Windows, PowerShell, telemetri Office | Bergabung ke domain agar event logon masuk ke tempat sentral |
| Linux server | Target aplikasi dan sumber syslog | Format log beragam untuk berlatih |
| Wazuh manager plus agen | Pengumpulan log sentral, aturan, dasbor | Gratis dan open source; gunakan dokumentasi resmi untuk langkah pemasangan terkini |
| VM penyerang (mis. Kali) | Menghasilkan telemetri serangan yang aman dan terkendali | Hanya digunakan terhadap lab ini |
| Jaringan host-only | Isolasi dari jaringan nyata Anda | Hindari adapter bridge; snapshot baseline-nya |

## Menghasilkan Telemetri Sendiri dengan Aman

Atomic Red Team adalah proyek open source berisi teknik uji kecil-kecilan, dipetakan ke MITRE ATT&CK, yang dirancang untuk dijalankan terhadap sistem yang Anda miliki guna memeriksa apakah deteksi Anda menyala. Alur kerja pertahanannya: pilih teknik yang seharusnya dicakup aturan Anda, jalankan ujiannya di lab, konfirmasi alert dan timeline muncul, lalu perbaiki aturan jika tidak ada yang menyala. Ini adalah validasi pertahanan yang sah di mesin Anda sendiri — verifikasi repositori terkini dan catatan keamanannya sebelum menjalankan apa pun, nonaktifkan ujian yang tidak Anda pahami, dan jangan pernah mengarahkan tool uji ke sistem di luar lab Anda.

## Range dan Platform Gratis

| Platform | Biaya | Format | Fokus |
|---|---|---|---|
| LetsDefend | Tier gratis plus berbayar | Lingkungan SOC ter-hosting | Triage alert dan investigasi pada alert simulasi |
| Blue Team Labs Online | Tier gratis plus berbayar | Lab berbasis browser | Tantangan investigasi dan analisis artifact |
| CyberDefenders | Tier gratis plus berbayar | Lab bergaya CTF blue team | Tantangan forensics dan analisis |
| TryHackMe (ruang blue team) | Tier gratis plus berbayar | Ruang berbasis browser terpandu | Jalur belajar terstruktur untuk pembela |
| SANS CyberRanges / NetWars | Bervariasi (sering melalui acara atau kursus SANS) | Range berbasis instruktur atau acara | Pembangunan keterampilan terstruktur dan intensif |

Verifikasi penawaran terkini, batas tier gratis, dan harga di situs resmi masing-masing platform — hal ini sering berubah.

## CTF Pertahanan

CTF memampatkan keterampilan di bawah tekanan waktu: tantangan forensics membangun kelancaran artifact, tantangan deteksi membangun logika kueri, dan skenario insiden membangun prioritisasi. Perlakukan write-up sebagai bahan portofolio — catat artifact mana yang Anda gunakan, pertanyaan apa yang dijawabnya, dan apa yang akan Anda otomatiskan kali berikutnya. Peringkat kurang penting daripada write-up-nya; pembelajaran terjadi saat Anda menjelaskan jalur Anda.

## Rencana Lab 30 Hari

| Hari | Fokus | Keluaran |
|---|---|---|
| 1-7 | Bangun dan snapshot lab; pasang Wazuh; daftarkan agen | Dasbor yang menampilkan event nyata dari tiga mesin |
| 8-14 | Pelajari sumber log Anda: event logon Windows, logging PowerShell, log autentikasi Linux | Cheat sheet satu halaman berisi event yang benar-benar Anda gunakan |
| 15-21 | Jalankan 5-10 uji atomic; sesuaikan aturan yang tetap diam atau menyala terlalu banyak | Lima deteksi yang sudah disesuaikan beserta catatan |
| 22-30 | Latihan penuh: suntik sebuah insiden dan tangani dari awal hingga akhir | Timeline, laporan insiden singkat, daftar pelajaran |

## Catatan Keamanan

Jalankan segala sesuatu di dalam lab terisolasi Anda: jaringan host-only, snapshot sebelum setiap latihan, dan tidak ada aktivitas uji terhadap sistem mana pun yang tidak Anda miliki atau tidak memiliki otorisasi tertulis untuk disentuh. Perlakukan telemetri serangan yang dihasilkan sebagai data latihan bagi pertahanan Anda sendiri — tidak lebih dari itu.

## Lihat juga

- [Tool Pertahanan](defense-tooling.md)
- [Karier & Sertifikasi Blue Team](blue-team-career.md)
- [Threat Hunting](../detection/threat-hunting.md)
- [Triage Alert](../response/triage-alerts.md)
