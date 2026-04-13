<h1>Queue Tree + Limit Profil MikroTik RoS7</h1>
Penggunaan skrip ini mengharuskan Anda menghapus seluruh konfigurasi sebelumnya, terutama yang masih menggunakan Simple Queue. Pastikan Anda telah melakukan backup konfigurasi sebelum melanjutkan.
<br><br>Berikut adalah langkah-langkahnya:

- Penandaan QoS (Quality of Service)

```shell
/ip firewall mangle
add chain=forward protocol=udp dst-port=53 action=mark-packet new-packet-mark=pkt_dns passthrough=no comment="Priority 1: DNS"
add chain=forward protocol=icmp action=mark-packet new-packet-mark=pkt_icmp passthrough=no comment="Priority 1: ICMP/Ping"
add chain=forward protocol=udp dst-port=3478,45395,50318,59234 action=mark-packet new-packet-mark=pkt_wa passthrough=no comment="Priority 3: WA Call"
add chain=forward protocol=udp dst-port=!443 action=mark-packet new-packet-mark=pkt_game passthrough=no comment="Priority 2: Game Online"
add chain=forward protocol=tcp dst-port=80,443 action=mark-packet new-packet-mark=pkt_browsing passthrough=no comment="Priority 4: Browsing"
add chain=forward protocol=udp dst-port=443 action=mark-packet new-packet-mark=pkt_browsing passthrough=no comment="Priority 4: Streaming/QUIC"
add chain=forward action=mark-packet new-packet-mark=pkt_other passthrough=no comment="Priority 8: Download/Others"
```

- Konfigurasi Queue Type
```shell
/queue type
add kind=pcq name=pcq-down pcq-classifier=dst-address pcq-rate=0
add cake-overhead=18 kind=cake name=cake-pppoe
```

- Konfigurasi Queue Tree
```shell
/queue tree
add name="00. TOTAL-GLOBAL" parent=global max-limit=290M
add name="1. DNS" parent="00. TOTAL-GLOBAL" packet-mark=pkt_dns priority=1 queue=pcq-down
add name="2. ICMP" parent="00. TOTAL-GLOBAL" packet-mark=pkt_icmp priority=1 queue=pcq-down
add name="3. WHATSAPP" parent="00. TOTAL-GLOBAL" packet-mark=pkt_wa priority=3 queue=pcq-down limit-at=30M max-limit=290M
add name="4. GAME" parent="00. TOTAL-GLOBAL" packet-mark=pkt_game priority=2 queue=pcq-down limit-at=50M max-limit=290M
add name="5. BROWSING" parent="00. TOTAL-GLOBAL" packet-mark=pkt_browsing priority=4 queue=pcq-down limit-at=100M max-limit=290M
add name="6. LAINNYA" parent="00. TOTAL-GLOBAL" packet-mark=pkt_other priority=8 queue=pcq-down max-limit=290M
```
Catatan: <br>
Sesuaikan nilai max-limit=290M dengan total bandwidth yang Anda miliki.
Sebagai contoh, jika total bandwidth adalah 300 Mbps, disarankan untuk tidak menggunakan seluruh kapasitas tersebut. Sisakan sebagian bandwidth guna menjaga stabilitas jaringan dan menghindari terjadinya bottleneck.
<br><br>

<h1>Langkah Akhir: Konfigurasi Manual di Winbox (PENTING!)</h1>
Skrip di atas telah mengatur jalur utama serta prioritas trafik secara global. Tahap selanjutnya adalah mengelola pembatasan pada sisi pelanggan.

Buka Winbox Anda dan lakukan dua hal ini:

1. Seting PPP Profile untuk Pelanggan (Kecepatan Paket Berjenjang)
- Masuk ke menu PPP > Profiles, klik/buat profil (misal: Paket-10M). Di tab Limits:
- Rate Limit (rx/tx): Masukkan kecepatan paketnya, misal 10M/10M
- Parent Queue: Pilih none (agar profil berdiri sendiri)
- Queue Type: Pilih cake-pppoe (untuk menjaga latency tetap stabil di sisi pelanggan)
- Insert Queue Before: Pilih bottom (agar tidak mengganggu konfigurasi prioritas lainnya)

2. Seting Hotspot User Profile (Jika Jual Voucher)
- Masuk ke menu IP > Hotspot > User Profiles, pilih profil voucher Anda.
- Rate Limit (rx/tx): Masukkan limit vouchernya, misal 3M/3M
- Queue Type: Pilih default atau pcq-down (Untuk hotspot standar sudah cukup)
- Insert Queue Before: Pilih bottom

