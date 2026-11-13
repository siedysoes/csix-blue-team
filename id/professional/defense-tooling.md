# Tool Pertahanan

Toolkit seorang pembela mencakup deteksi (SIEM, EDR, sensor jaringan), investigasi (forensics, manajemen kasus), dan otomasi — dan sebagian besar bersifat gratis atau open source. Artikel ini mengelompokkan opsi-opsi yang umum berdasarkan kategori dan, yang lebih penting, berdasarkan kapan menggunakannya. Tool gratis tetap membutuhkan waktu untuk dideploy dan disesuaikan, jadi pilih yang secara realistis Anda mampu operasikan.

## SIEM / XDR

| Tool | Apa itu | Kapan digunakan |
|---|---|---|
| Wazuh | SIEM dan XDR gratis, open source: agen endpoint, ingestion log, aturan, dasbor | Visibilitas self-hosted untuk lab dan lingkungan kecil hingga menengah |
| Elastic Security | Dibangun di atas Elastic Stack; pencarian log, aturan deteksi, dasbor dalam satu paket | Anda sudah menjalankan Elastic dan menginginkan pencarian yang ketat plus deteksi bersamaan |
| Splunk (tier gratis) | Lisensi gratis Splunk Enterprise dengan batas volume harian | Belajar SPL dan pencarian gaya enterprise — verifikasi ketentuan tier gratis terkini |

## Endpoint: Alternatif EDR

| Tool | Apa itu | Kapan digunakan |
|---|---|---|
| Velociraptor | DFIR dan hunting endpoint berskala besar: kumpulkan artifact dan jalankan kueri di banyak host | Penentuan cakupan cepat, pengumpulan artifact, hunting seluruh armada |
| osquery | Kueri endpoint dengan SQL (`SELECT ... FROM processes`) | Visibilitas ringan lintas platform; kueri terjadwal bergaya kepatuhan |

Keduanya menambah visibilitas endpoint tetapi bukan pengganti penuh yang langsung dapat dipasang untuk respons langsung EDR komersial — verifikasi fitur terkini terhadap kebutuhan Anda.

## Deteksi Jaringan

| Tool | Apa itu | Kapan digunakan |
|---|---|---|
| Suricata | IDS/IPS open source: deteksi berbasis signature pada trafik | Deteksi atau blokir trafik berbahaya yang diketahui di perimeter atau sebuah segmen |
| Zeek | Metadata dan log jaringan (koneksi, DNS, HTTP, TLS) tanpa signature | Investigasi dan deteksi berbasis perilaku saat signature meleset |

Suricata memberi tahu Anda "ini cocok dengan sebuah aturan"; Zeek memberi tahu Anda "apa yang terjadi di jaringan". Bersama-sama keduanya memenuhi kebutuhan signature dan metadata.

## Penyaringan DNS dan Proxy

| Tool | Apa itu | Kapan digunakan |
|---|---|---|
| Pi-hole | DNS sinkhole seluruh jaringan dengan dasbor | Lab rumah dan jaringan kecil; umpan balik cepat tentang domain berbahaya |
| blocky / CoreDNS dengan plugin | Proxy dan resolver DNS yang dapat dikonfigurasi dengan blocklist dan logging | Lingkungan yang menginginkan penyaringan DNS yang dapat diskrip |

Pemblokiran DNS adalah containment yang murah dan dini — periksa apa yang tim jaringan Anda izinkan di lingkungan nyata.

## Hunting dan Forensics

| Tool | Apa itu | Kapan digunakan |
|---|---|---|
| TheHive + Cortex | Manajemen kasus keamanan open source (TheHive) dengan pekerja analisis observable (Cortex) | Mengubah alert menjadi kasus bersama dan mengotomatiskan lookup |
| Autopsy | Platform forensik disk: timeline, pencarian kata kunci, analisis file | Penelusuran mendalam pada sebuah disk image |
| Plaso (log2timeline) | Pembuat super-timeline dari banyak sumber artifact | Membangun lintas-artifact timeline dengan cepat |

## Penipuan dan Otomasi

- **Honeytoken:** artifact palsu tetapi masuk akal — kredensial jebakan di sebuah repositori, share admin palsu, akun pengguna jebakan — yang tidak boleh pernah disentuh. Setiap akses merupakan sinyal kuat dengan false positive yang hampir nol. Deploy mereka, beri alert untuk mereka, dan jangan pernah membuatnya menjadi nyata.
- **SOAR / orkestrasi (mis. Shuffle):** hubungkan tool agar langkah triage yang berulang berjalan otomatis — enrichment IP, membuka kasus, mengirim ke sebuah kanal. Otomatiskan pengulangan, bukan penilaian; jaga tindakan yang tidak dapat dibatalkan di balik persetujuan manusia.

## Tool Mana untuk Pekerjaan Mana

| Anda perlu... | Pilihan pertama | Mengapa |
|---|---|---|
| Sentralisasi log dan alert di atasnya | Wazuh atau Elastic Security | Gratis, self-hosted, berbasis aturan |
| Menanyakan riwayat satu host dengan cepat | osquery atau Velociraptor | Kueri endpoint langsung tanpa EDR penuh |
| Mengumpulkan evidence berskala besar | Velociraptor | Pengumpulan artifact bertarget di banyak host |
| Mendeteksi trafik known-bad secara inline | Suricata (mode IPS) | Signature dengan pemblokiran |
| Memahami apa yang dilakukan sebuah host di jaringan | Log Zeek | Metadata kaya untuk hunting |
| Memblokir domain berbahaya dengan murah | Pi-hole (rumah) / penyaringan DNS (organisasi) | Containment dini dengan usaha rendah |
| Mengoordinasikan sebuah insiden bersama tim | TheHive plus SIEM Anda | Status kasus dan evidence bersama |
| Mengotomatiskan triage yang berulang | Shuffle atau skrip Anda sendiri | Langkah manual lebih sedikit |
| Segera tahu ketika kredensial palsu digunakan | Honeytoken | Alerting dengan sinyal kuat |

## Catatan Pemasangan

Semua tool di atas memiliki dokumentasi resmi — mulai dari sana dan verifikasi versi terkini, platform yang didukung, dan panduan konfigurasi sebelum deploy. Rancang dulu (apa yang akan Anda pantau, ke mana data mengalir, siapa yang membaca alert), baru pasang: tool yang sudah dideploy tetapi tidak disesuaikan hanyalah beban. Perlakukan perintah quick-start apa pun yang Anda temukan online sebagai tidak tepercaya sampai diperiksa terhadap dokumentasi resmi.

## Aturan Emas

Tool hanya membela apa yang benar-benar Anda pantau dan sesuaikan. Cakupan mengalahkan jumlah: lima sumber log yang Anda perhatikan setiap hari mengalahkan lima puluh integrasi yang tidak pernah dikueri siapa pun. Setiap tool memerlukan penanggung jawab, jadwal penyesuaian, dan alert yang ditindaklanjuti lewat tiket atau halaman oleh seseorang — jika tidak, tool itu menghasilkan data, bukan pertahanan.

## Lihat juga

- [Dasar SIEM](../fundamentals/siem-basics.md)
- [Pertahanan Jaringan](../fundamentals/network-defense.md)
- [Lab & Range Pertahanan](defense-labs.md)
- [Menulis Aturan Sigma](../detection/sigma-rules.md)
