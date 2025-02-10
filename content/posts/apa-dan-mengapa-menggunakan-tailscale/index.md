---
date: '2025-02-09T22:57:35+07:00'
draft: false
title: "Apa & Mengapa menggunakan Tailscale"
tags: ["Vps"]
categories: ["DevOps"]
ShowToc: true
TocOpen: false
summary: "Zero Trust networks yang menjadi salah satu alasan mengapa Tailscale ada."
weight: 2
---
## Komunikasi Antar Komputer
Untuk dapat membuat komputer berkomunikasi satu sama lain, satu-satunya cara adalah melalui alamat IP sebagai pengenal. Hal ini berbeda dengan manusia yang bisa berkomunikasi dengan memanggil nama, nama orang tua, atau bahkan kekurangannya.

## Cara Mendapatkan Alamat IP
Ada dua cara untuk mendapatkan alamat IP:

1. **Statis**: Kita atur sendiri alamat IP-nya.
2. **Dinamis**: Ada sesuatu yang mengaturnya secara otomatis, yaitu **DHCP**.

Namun, tidak sesederhana itu. Jika kita menyalakan dua komputer dan masing-masing sudah diset dengan alamat IP yang berbeda, apakah mereka langsung bisa berkomunikasi? Tentu tidak.

## Media Komunikasi
Dalam komunikasi, tidak hanya diperlukan "lawan bicara," tetapi juga "media." Contohnya, jika kita ingin *say hello* kepada seseorang, media yang digunakan adalah mulut untuk berbicara (*send*) dan telinga untuk mendengarkan (*receive*).

Di dunia komputer, media komunikasi ini bisa berupa:

- **Hub**
- **Switch**
- **Router**

## Protokol Komunikasi
Setelah ada pengenal (alamat IP) dan media, apakah komunikasi langsung bisa terjadi? Belum tentu. Satu hal lagi yang dibutuhkan adalah **protokol**, yaitu aturan komunikasi.

Misalnya, komunikasi akan kurang lancar jika seseorang dari Bandung berbicara dengan orang Surabaya menggunakan bahasa Sunda.

### Contoh Protokol: ICMP
Protokol adalah tentang kesepakatan. Contoh sederhana dalam dunia komputer adalah penggunaan protokol **ICMP**. Untuk mengetahui apakah sebuah perangkat bisa dihubungi, biasanya digunakan perintah `PING`. Jika perangkat target mengenali `PING` tersebut, maka ia akan membalas dengan `PONG` untuk memberi tahu bahwa perangkat tersebut dapat dihubungi.

Dengan protokol ICMP, komunikasi sederhana antar perangkat menjadi mungkin dan lebih terstruktur.

{{< figure src="images/contoh-sederhana-proses-ping-pada-router.png" width="400" align="center">}}

**Router** berfungsi sebagai "media" untuk komunikasi antar jaringan.

Kasus di atas berlaku untuk komunikasi yang berada di jaringan lokal atau yang biasa disebut dengan **Local Area Network (LAN)**, yaitu jaringan yang terbatas di area lokal yang relatif sempit.

## Era Internet Saat Ini
Sekarang adalah era internet. Internet sederhananya adalah jaringan komputer yang saling terhubung. Komputer yang berada di Bandung dapat berkomunikasi dengan komputer yang berada di Singapura melalui jaringan internet. Media yang digunakan dalam komunikasi tersebut masih router, tetapi dengan konfigurasi yang jauh lebih kompleks.

{{< figure src="images/proses-komunikasi-ping-antar-negara.png" align="center">}}

Contoh yang Disederhanakan: Melakukan `PING` dari `192.168.10.6` ke `1.1.1.1` di Singapore. Sebagai catatan tambahan, biasanya jaringan untuk level kota ke atas akan melalui **Point of Presence (PoP)**, dan untuk tingkat negara (kumpulan kota) ke atas akan melalui **Internet Exchange Point (IXP)**.

Bagian dari ilustrasi di atas yang tidak saya bahas adalah tentang **Network Address Translation (NAT)**, yang akan saya bahas nanti di tulisan khusus. Ilustrasi tersebut hanyalah gambaran sederhana; dalam praktiknya seharusnya lebih kompleks.

Terlihat bahwa untuk mencapai `1.1.1.1` dari `192.168.10.6`, komputer kita harus melalui perangkat dengan alamat IP `118.99.118.144` lalu `27.111.228.9`, yang semuanya adalah alamat IP publik. Sedangkan `192.168.10.6` adalah alamat IP privat.

Total alamat IP versi 4 yang ada adalah **4.294.967.296**. Dari jumlah tersebut, IP publik tersedia sekitar **3.706.452.992**. Bayangkan komputer kita terhubung dengan **3 miliar** komputer lainnya di internet.

## Keamanan Jaringan

Salah satu poin penting yang ingin saya *highlight* di sini adalah keamanan.

Devbox saya yang ada di Singapore secara teori dapat berkomunikasi dengan 3 miliar komputer lainnya yang ada di internet. Begitu pula sebaliknya, 3 miliar komputer tersebut juga bisa berkomunikasi dengan komputer saya.

Ingat analogi *mulut & telinga* dalam komunikasi?

Dalam komunikasi antar komputer, analogi "telinga" adalah **port**. Ketika komputer A mengirim `PING` ke komputer B, komputer A membuka port yang digunakan untuk "mendengarkan" balasan dari komputer B. Komputer B kemudian akan mengirimkan balasan tersebut ke port yang digunakan oleh komputer A.

{{< figure src="images/contoh-ping-pada-komputer.png" width="600" align="center">}}

`PING` dari komputer A yang menggunakan port **55** dan komputer B yang menggunakan port **28**. Terdapat sekitar **64k port** yang sudah diketahui, dengan total maksimal **65.535** port yang dapat dibuka. Port yang paling sering digunakan untuk berkomunikasi adalah **22**, yakni untuk membuat koneksi **SSH** alias mengakses komputer dari jarak jauh.

Beberapa orang ada yang menyamarkan port untuk SSH, misal dari **22** menjadi **24** demi menghindari serangan. Namun pendekatan tersebut adalah **"security by obscurity"** karena pada akhirnya penyerang pun akan menemukannya berkat alat seperti `nmap(1)`.

Mari kita fokus terkait SSH terlebih dahulu.

Masalahnya adalah bagaimanapun port tersebut tetap terbuka ke jaringan publik. Mungkin masalah ini dapat diatasi dengan melakukan **rate limit** di **firewall** untuk menghindari serangan **bruteforce**, namun tetap tidak menyelesaikan masalah tersebut.

Solusi yang paling umum digunakan adalah dengan membuat jaringan pribadi (virtual) di atas jaringan publik atau yang biasa disebut dengan **Virtual Private Network (VPN)**.

{{< figure src="images/contoh-vpn-gateway.png" width="600" align="center">}}


Ilustrasi sederhana tentang VPN dengan subnet **10.0.0.0/24**. Sekilas terlihat seperti komunikasi dalam jaringan seperti biasa, bukan? Dari gambaran di atas, paket yang dikirimkan untuk "jaringan biasa" sederhananya seperti ini:

**IP header (inner, encrypted):**
- source: `192.168.6.8`
- destination: `192.168.7.9`

**IP header (outer):**
- source: `192.168.6.8`  
- destination: `192.168.7.9`

Namun dalam jaringan VPN, sederhananya seperti ini:

**IP header (inner, encrypted):**
- source: `10.0.0.2`  
- destination: `10.0.0.3`

**IP header (outer):**
- source: `<ip_public_a>`  
- destination: `<ip_public_b>`

Yang sederhananya, router (dan siapapun yang berada di balik itu) tidak mengetahui siapa berkomunikasi dengan siapa, tapi yang dia ketahui adalah paket tersebut memang untuk dia dan dia tahu harus diteruskan ke mana paket tersebut.

{{< figure src="images/contoh-vpn-gateway-kompleks.png" width="600" align="center">}}

IP yang ada di atas kotak komputer adalah IP publik milik router. VPN pada umumnya menggunakan pendekatan **"hub-and-spoke"** atau yang biasa dikenal dengan **"star network"**, yang sederhananya mengandalkan satu gerbang sebagai **relay**. Alasannya beragam, tetapi yang paling umum terkait dengan firewall (yang akan dibahas lebih lanjut di tulisan tentang NAT).

Misal kita ingin membuat koneksi SSH dari komputer A ke komputer B melalui VPN. Karena mereka tersambung ke VPN, pasti mereka memiliki **Virtual Network Interface** tambahan yang ditujukan untuk tunneling dengan alamat IP yang sudah didefinisikan. Anggap network interface tersebut bernama **utun3** (nama dapat bervariasi tergantung *mood* kernel).

Ketika pengguna menjalankan perintah:

```bash
ssh faultable@10.0.0.3
```

Dalam routing table, paket tersebut akan di-*handle* oleh **utun3**. Dari utun3, paket terenkripsi dikirim ke default/external router dengan tujuan **143.198.198.198**, yaitu alamat IP publik dari komputer B.

Dari mana komputer A (utun3) tahu bahwa paket untuk **10.0.0.3** adalah komputer dengan IP publik **143.198.198.198**? Ada banyak cara, misalnya melalui **VPN Gateway**.

Paket kemudian dikirim oleh ISP A ke VPN Gateway, yang meneruskannya ke ISP B. ISP B mengetahui bahwa paket tersebut untuk dirinya dan harus diteruskan, misalnya ke port **22**.

Berdasarkan kasus di atas, kita bisa memitigasi serangan SSH dengan hanya menerima koneksi ke port **22** dari alamat IP VPN yang sudah didefinisikan.

Berkat penggunaan VPN, dapat diasumsikan bahwa segala sesuatu di jaringan tersebut terjamin dan aman. Namun, inilah alasan VPN memiliki moto **"trust but verify"**. Untuk terhubung ke VPN, pengguna biasanya harus melakukan autentikasi terlebih dahulu, dan paket selalu berada dalam bentuk terenkripsi.

Misalnya, ada tiga komputer yang terhubung ke VPN. Jika salah satu komputer diretas dan seseorang dapat mengakses mesin tersebut, koneksi dari komputer tersebut ke perangkat lain dalam VPN tetap dianggap aman.

Karena moto VPN adalah *"trust but verify"*, maka si komputer C yang diretas dapat berkomunikasi dengan komputer lain, termasuk melakukan SSH. Meski penyerang mungkin tidak mengetahui kredensial target, ini tetap menjadi risiko karena koneksi dalam jaringan VPN dianggap terjamin dan aman.

Berapa lama waktu yang dibutuhkan untuk *crack* SSH private key menggunakan `john the ripper`? Mungkin tidak selamanya.

Selain keamanan, VPN memungkinkan akses ke komputer yang tidak berada dalam satu jaringan fisik tetapi tersembunyi di balik alamat IP dinamis. Contohnya, mengakses **Raspberry Pi** dari Starbucks.

Pada intinya, VPN dapat membuat jaringan "interlokal" terasa seperti jaringan lokal. Bayangkan jika ada fitur seperti **AirDrop®-like experience** untuk mengirim berkas dengan mudah dan aman secara **peer-to-peer** menggunakan VPN.

## Wireguard®

Wireguard adalah program untuk membuat VPN. Program ini bersumber kode terbuka, relatif sederhana, dan cepat. Autentikasi di Wireguard menggunakan **public key cryptography** seperti halnya SSH.

Paket yang ditukar di Wireguard menggunakan protokol UDP, berbeda dari kebanyakan program VPN lainnya seperti OpenVPN dan tinc yang menggunakan TCP. Ada pro dan kontra dalam pemilihan protokol UDP karena ISP sering memblokir protokol UDP untuk **non well-known port**. Namun, hal ini bukan masalah besar karena kita dapat melakukan **hole punching** (yang akan dibahas di tulisan tentang NAT).

Wireguard memungkinkan komunikasi antar komputer secara *peer-to-peer*, berbeda dari VPN lain yang biasanya menggunakan pendekatan *hub-and-spoke* yang berguna dalam proses autentikasi. Karena Wireguard menggunakan **PKI (Public Key Infrastructure)** dalam autentikasi, hanya ada beberapa konfigurasi yang perlu didefinisikan:

- **PrivateKey:** private key dari peer ini
- **Address:** alamat IP dari peer ini
- **Endpoint:** lokasi peer tersebut
- **PublicKey:** public key dari peer tersebut
- **AllowedIPs:** kumpulan alamat IP yang ingin diarahkan melalui Wireguard

### Contoh Konfigurasi Wireguard®

Misalnya, konfigurasi di komputer A:

```ini
[Interface]
PrivateKey = <private_key_kita>
Address = 10.0.0.3/24

[Peer]
PublicKey = <public_key_peer>
AllowedIPs = 10.0.0.0/24
Endpoint = 143.198.198.198:1337
```

Ketika kita melakukan `PING` ke `10.0.0.3`, paket tersebut akan dikirimkan melalui interface yang dibuat oleh Wireguard ke peer yang berada di `143.198.198.198`.

Peer yang berada di IP `143.198.198.198` pun harus memiliki konfigurasi yang sesuai, misalnya:

```ini
[Interface]
PrivateKey = <private_key_peer>
Address = 10.0.0.1/24

[Peer]
PublicKey = <public_key_kita>
AllowedIPs = 10.0.0.2/32
```

Bagian **Endpoint** opsional jika komputer berada di balik IP dinamis. Sayangnya, rata-rata komputer pribadi memang menggunakan alamat IP dinamis. Untuk dapat terhubung ke peer lain, kita memerlukan **Hub sebagai STUN server** agar dapat mengetahui alamat IP publik peer.

Selain itu, penting untuk mengelola public dan private key dengan baik. Jika ada 8 komputer yang ingin terhubung melalui Wireguard secara *peer-to-peer*, maka akan ada \(n(n-1)\) koneksi yang terjadi. Jika \(n = 8\), maka ada 56 koneksi yang terbentuk.

{{< figure src="images/ilustrasi-jelek-koneksi-p2p.png" width="500" align="center">}}

ilustrasi jelek koneksi p2p Setiap peer harus memiliki public key dari peer yang ingin tersambung. Setup VPN terakhir saya melalui Wireguard menggunakan pendekatan hub-and-spoke karena… lebih sederhana. 

{{< figure src="images/ilustrasi-bagus-p2p.png" width="500" align="center">}}

Dengan setup seperti di atas, ketika komputer 2 ingin mengakses komputer 3, paket dikirimkan melalui komputer 1 terlebih dahulu. Namun, jika komputer 1 mengalami gangguan atau tidak aktif, koneksi antar komputer lain akan terputus.

Konfigurasi ini cukup sederhana. Komputer selain komputer 1 bertindak sebagai *client*, sementara komputer 1 memegang semua public key yang ada serta bertanggung jawab dalam menentukan apakah komputer 2 boleh mengakses komputer 3.

Masalah utama yang sering saya rasakan adalah ketidakandalan komputer 1 yang menyebabkan koneksi VPN menjadi tidak efektif. Karena masalah ini, saya mencoba solusi lain yaitu ZeroTier. Namun, meski menggunakan pendekatan mesh network, ZeroTier terasa terlalu kompleks dari sisi konfigurasi dan kurang *reliable* terutama dalam mode “full tunnel” karena paket sering kali tidak dikirim melalui interfacenya.

## Tailscale Sebagai Solusi Praktis
Tailscale adalah program untuk membuat VPN yang dibangun di atas WireGuard. Segala manfaat yang ada pada WireGuard juga hadir di Tailscale. Namun, Tailscale melengkapi beberapa kekurangan WireGuard yang bukan merupakan tanggung jawab WireGuard itu sendiri, seperti:

1. **Mesh network:** Meskipun WireGuard akan mendukungnya melalui [wg-dynamic](https://github.com/WireGuard/wg-dynamic/blob/master/docs/idea.md), Tailscale sudah menyediakan dukungan secara bawaan.
2. **Machine authentication:** Jika di WireGuard hanya menggunakan public/private key, Tailscale menyediakan autentikasi tambahan seperti SSO.
3. **Magic DNS:** Mirip dengan mDNS, namun lebih fleksibel. Di WireGuard, kita harus secara manual mengatur DNS untuk mendapatkan alamat IP private mesin lain.
4. **ACL:** Jika di WireGuard konfigurasi ACL dilakukan melalui *AllowedIPs*, di Tailscale ACL dapat diatur menggunakan format JSON, memudahkan pengaturan seperti membatasi SSH hanya untuk grup tertentu.

Tailscale menjanjikan instalasi tanpa konfigurasi (*zero config*), sehingga bahkan orang tanpa latar belakang IT pun dapat dengan mudah mengakses layanan privat dalam jaringan VPN. Contohnya, pacar saya yang bukan orang IT tetap bisa mengakses layanan privat tanpa harus memahami *public key*, *endpoint*, *allowed IPs*, dan konfigurasi lainnya.

### Cara Memulai Menggunakan Tailscale
Autentikasi di Tailscale saat ini mendukung akun Google, Microsoft, dan penyedia SSO lainnya. Saya belum coba apakah integrasi dengan solusi seperti Keycloak bisa berjalan, tapi jika Tailscale mendukung SAML, seharusnya bisa.

Langkah memulainya cukup sederhana:
1. Unduh aplikasi Tailscale untuk sistem operasi yang digunakan.
2. Lakukan proses autentikasi.
3. Selesai.

### Mengapa Menggunakan Tailscale?
Sejujurnya, saya tertarik untuk menulis lebih dalam tentang alasan penggunaan Tailscale, khususnya terkait *Zero Trust networks*, yang menjadi salah satu alasan utama keberadaan Tailscale. Karena topik ini cukup menarik, saya akan membuat tulisan terpisah sebagai bagian kedua dari tulisan ini.

## Penutup
Saya rasa tulisan ini bisa menjadi gambaran awal untuk tulisan-tulisan selanjutnya di blog ini. Di postingan berikutnya, saya akan membahas tentang NAT dan kemudian berfokus pada dunia *peer-to-peer* yang menjadi topik menarik buat saya pelajari.

