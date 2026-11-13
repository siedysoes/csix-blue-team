# Pertahanan Jaringan

Pertahanan jaringan adalah praktik menumpuk lapisan kontrol di sekitar lalu lintas sehingga satu kegagalan tunggal tidak berubah menjadi breach, serta mengawasi lalu lintas tersebut agar Anda menyadari saat sesuatu lolos. Pertahanan ini melengkapi pertahanan endpoint dan berbasis log: bahkan ketika Anda tidak bisa membaca isi sebuah koneksi, biasanya Anda tetap bisa melihat bahwa koneksi itu terjadi. Halaman ini membahas model berlapis, dasar firewall serta IDS/IPS, segmentasi, pemblokiran dini, dan pemantauan berbasis flow.

## Model Berlapis

| Lapisan | Komponen | Tugas utama |
| --- | --- | --- |
| Perimeter | Firewall stateful | Mengizinkan lalu lintas yang sah masuk dan keluar, menolak semua sisanya secara default |
| Internal | Segmentasi (VLAN, ACL, kebijakan zero trust) | Menahan pergerakan, agar workstation yang terkompromi tidak bisa menjangkau semua server |
| Inspeksi | IDS / IPS | Mendeteksi atau menghentikan pola yang sudah diketahui berbahaya dalam lalu lintas |
| Pemblokiran dini | DNS filtering | Menolak meresolusi domain berbahaya sebelum koneksi apa pun dicoba |
| Mediasi | Proxy / secure web gateway | Mengontrol dan menginspeksi permintaan web keluar |
| Visibilitas | NetFlow, sFlow, deteksi jaringan | Mencatat siapa bicara dengan siapa, seberapa banyak, dan kapan — dasar untuk threat hunting |

Setiap lapisan gagal dengan cara yang berbeda, dan justru itulah intinya: penyerang yang melewati satu lapisan tetap bertemu lapisan berikutnya, dan setiap lapisan meninggalkan jejak.

## Dasar Firewall

**Stateful inspection** adalah perilaku standar firewall modern: firewall melacak status koneksi seperti new, established, dan related, serta secara otomatis mengizinkan lalu lintas balasan untuk koneksi yang firewall izinkan sendiri, alih-alih membutuhkan aturan terpisah untuk setiap paket balasan.

Dua model operasi yang penting:

- **Explicit allow (default deny)** — tidak ada yang lolos kecuali sebuah aturan menyebutnya secara eksplisit. Inilah model yang Anda inginkan di sebuah trust boundary.
- **Allow-all** — aturan seperti `any -> any, accept` membuat firewall menjadi sekadar dekorasi: tidak memblokir apa pun dan tidak mencatat apa pun yang berguna.

**Prinsip kebersihan aturan:**

- Satu aturan, satu tujuan, masing-masing dengan pemilik dan komentar yang menjelaskan alasannya.
- Tidak ada izin `any/any` di batas; hapus aturan yang tertimpa (shadowed) oleh aturan sebelumnya atau sudah tidak punya justifikasi bisnis.
- Catat baik izin maupun penolakan di batas, dan tinjau log penolakan untuk upaya reconnaissance.
- Uji perubahan di jendela pemeliharaan, siapkan rollback, dan sertifikasi ulang set aturan secara berkala.

## IDS vs IPS

| Aspek | IDS (intrusion detection) | IPS (intrusion prevention) |
| --- | --- | --- |
| Penempatan | Tap atau mirror port (out of band), melihat salinan lalu lintas | Inline, lalu lintas mengalir secara fisik melalui perangkat |
| Aksi | Hanya alert; manusia yang memutuskan apa yang terjadi selanjutnya | Alert dan bisa menjatuhkan (drop) atau mereset koneksi secara otomatis |
| Risiko utama | Keterlewatkan atau reaksi terlambat terhadap alert | False positive memblokir lalu lintas bisnis yang sah |
| Paling cocok untuk | Visibilitas dan penyetelan di mana ketersediaan lebih diutamakan daripada kecepatan | Lingkungan dengan aturan yang sudah baik disetel dan pengecualian yang sudah diuji |

Keduanya bergantung pada dua pendekatan deteksi, sering kali dalam kombinasi:

- **Signature matching** membandingkan lalu lintas dengan pola yang sudah diketahui. Cepat dan tepat untuk ancaman yang sudah dikenal, tetapi buta terhadap hal baru.
- **Anomaly detection** membandingkan lalu lintas dengan baseline perilaku normal yang dipelajari. Bisa menandai aktivitas yang belum pernah terjadi, tetapi menghasilkan lebih banyak kebisingan dan butuh penyetelan.

## Segmentasi Jaringan

Sebuah **jaringan flat** (datar), yaitu satu broadcast domain di mana setiap host bisa menjangkau host lainnya, mengubah satu laptop yang terkompromi menjadi jalur menuju domain controller, basis data, dan sistem backup. Penyerang tidak membutuhkan teknik canggih ketika rutenya sudah terbuka.

Segmentasi menutup rute-rute tersebut: VLAN workstation, zona server, jaringan manajemen terpisah, serta aturan firewall atau ACL eksplisit di antaranya, dengan default deny dan hanya port yang diperlukan yang diizinkan — misalnya workstation menjangkau application server di port 443 dan tidak ada yang lain.

Framing **zero trust** melangkah lebih jauh: jangan pernah memberikan akses hanya berdasarkan lokasi jaringan. Verifikasi identitas dan kesehatan perangkat secara eksplisit, berikan hak akses paling minim (least privilege) per aplikasi, dan asumsikan jaringan itu sendiri bersifat permusuhan. Dalam praktiknya, segmentasi ditambah identitas yang kuat adalah titik awal sebagian besar organisasi.

## DNS Filtering dan Proxy sebagai Blokir Dini

Sebagian besar malware tidak bisa bergerak sampai ia meresolusi domain command-and-control (C2). Memblokir lookup tersebut di resolver atau proxy menghentikan rantai callback sebelum payload sempat terhubung — kontrol yang murah, dikelola terpusat, dan tidak membutuhkan perubahan apa pun di endpoint:

```
malware.exe  ->  DNS query: update-cdn.badexample[.]net
                        |
              resolver policy: known-bad domain
                        |
              NXDOMAIN / block page returned
                        |
        no connection established, no C2 session
                        |
       EDR and DNS logs record the attempted lookup
```

DNS filtering dan proxy adalah blokir dini, bukan blokir yang lengkap: penyerang memakai alamat IP langsung, domain yang dihasilkan secara algoritmik, dan DNS terenkripsi untuk menghindarinya. Perlakukan keduanya sebagai satu lapisan yang sekaligus menghasilkan telemetri bernilai tinggi seperti domain yang jarang, ledakan (burst) lookup, dan nama DNS yang tidak biasa panjang sebagai indikasi tunneling, serta pasangkan dengan EDR dan pemantauan flow.

## Dasar Pemantauan Jaringan

**NetFlow dan sFlow** menghasilkan rekaman metadata, bukan tangkapan paket: sumber, tujuan, port, protokol, jumlah byte, dan timestamp untuk setiap percakapan. Biasanya itu sudah cukup untuk melihat:

- **Beaconing** — koneksi yang teratur dan berjarak sama ke satu host eksternal, tanda tangan C2 yang klasik.
- **Data exfiltration** — transfer keluar yang besar atau waktunya janggal, terutama ke tujuan yang tidak terduga.
- **Scanning** — satu host internal menghubungi banyak host atau port dalam waktu singkat.

Metadata diskalakan jauh lebih baik daripada tangkapan paket penuh: lebih kecil, disimpan lebih lama, dan menjawab sebagian besar investigasi yang dimulai dari pertanyaan "siapa bicara dengan siapa, dan kapan?"

## Catatan tentang Enkripsi

Lalu lintas modern dienkripsi secara bawaan dengan TLS, sehingga tanpa intersepsi Anda melihat **metadata, bukan isi**: tujuan dan sertifikat, waktu koneksi, serta volume byte, tetapi bukan path URL atau payload-nya. Itu tetap menjawab banyak pertanyaan, tetapi tidak semuanya.

Dua respons yang bisa diterapkan: lakukan inspeksi TLS di proxy di mana kebijakan dan hukum mengizinkannya, dengan menimbang privasi, persetujuan, dan batasan regulasi secara eksplisit; atau lewati inspeksi isi dan andalkan telemetri endpoint, karena EDR melihat proses yang mengirim permintaan, command line yang melahirkannya, dan file yang dijatuhkannya — yang sering kali lebih penting daripada byte paketnya.

## Defense in Depth

```
                           Internet
                              |
                     +-----------------+
                     |  Perimeter FW  |   default-deny, stateful
                     +-----------------+
                              |
              +---------------+---------------+
              |                               |
        [ DMZ: web, mail ]            [ Workstations ]
              |                               |
              |                      [ Segmentation:
              |                        VLANs + ACLs ]
              |                               |
              +-------- [ Core: AD, servers ] +
                              |
          +-------------------+-------------------+
          |                   |                   |
     [ IDS / IPS ]      [ DNS filtering ]     [ Proxy ]
          |                   |                   |
          +------- telemetry: logs and flows -----+
                              |
                       [ NetFlow + EDR ]
                              |
                            [ SIEM ]   correlation and alerting
```

Setiap lapisan sekaligus memblokir apa yang bisa diblokirnya dan menyumbangkan bukti ke atas: penolakan firewall, kueri DNS, peristiwa proxy, dan rekaman flow menjadi sumber log yang di-SIEM-korelasikan menjadi alert — persis materi yang dibahas di halaman-halaman fundamental tentang log dan SIEM.

## Kaitan dengan Topik Sebelumnya

Perangkat jaringan adalah sumber log kelas satu: log firewall, proxy, dan DNS dikumpulkan, dinormalisasi, dan dikorelasikan di SIEM, tempat mereka bergabung dengan telemetri endpoint dari EDR. Saat sebuah deteksi terpicu, aksi respons bisa datang dari lapisan mana pun — blok DNS, aturan firewall, atau isolasi host — itulah mengapa pertahanan jaringan dan security operations diajarkan bersama.

## Lihat juga

- [Sumber Log yang Penting](log-sources.md)
- [Dasar SIEM](siem-basics.md)
- [Perangkat Pertahanan](../professional/defense-tooling.md)
- [Threat Hunting](../detection/threat-hunting.md)
