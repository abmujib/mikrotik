# 🚀 WHATSAPP STATIC ROUTING MIKROTIK RoS7
Optimized WhatsApp Traffic Routing via Specific ISP Gateway
<p align="center"> <img src="https://img.shields.io/badge/MikroTik-RouterOS%20v7-blue?style=for-the-badge&logo=mikrotik" /> <img src="https://img.shields.io/badge/Static_Routing-WhatsApp-green?style=for-the-badge&logo=whatsapp" /> <img src="https://img.shields.io/badge/Network-Optimization-orange?style=for-the-badge&logo=cloudflare" /> </p>

# 📌 Deskripsi
Konfigurasi ini digunakan untuk:
- 📲 Mengarahkan trafik WhatsApp melalui ISP tertentu
- 🌐 Memisahkan jalur WhatsApp dari trafik umum
- ⚡ Mengoptimalkan kestabilan WhatsApp Call, Chat, dan Media
- 🛡️ Menandai domain/IP WhatsApp secara otomatis

# 🗂️ Features
- ✅ Static WhatsApp Domain Routing
- ✅ Dynamic WhatsApp Detection via RAW
- ✅ Routing Table Khusus
- ✅ Mangle Routing Mark
- ✅ Local Network Detection
- ✅ RouterOS v7 Compatible
- ✅ Multi ISP Ready

# 📥 Installation
1. Copy script step-by-step
2. Paste ke Terminal MikroTik
3. Ganti gateway sesuai ISP
4. Verifikasi address-list & route

# 🧩 Configuration Steps
# STEP 1 — LOCAL NETWORK ADDRESS LIST
📘 Fungsi: 
<p>Digunakan untuk mendeteksi seluruh trafik yang berasal dari jaringan internal.</p>

```shell
/ip firewall address-list add address=192.168.0.0/16 list=LOCAL comment="LOCAL IP"
/ip firewall address-list add address=172.16.0.0/12 list=LOCAL comment="LOCAL IP"
/ip firewall address-list add address=10.0.0.0/8 list=LOCAL comment="LOCAL IP"
```

# STEP 2 — FIREWALL RAW DETECTION
📘 Fungsi:
- wa.me → Shortlink WhatsApp
- .whatsapp. → Semua subdomain WhatsApp
- 1d timeout → Refresh otomatis
```shell
/ip firewall raw
add action=add-dst-to-address-list address-list=whatsapp_list address-list-timeout=1d chain=prerouting comment="WhatsApp Routing" content=wa.me dst-address-list=!LOCAL src-address-list=LOCAL
add action=add-dst-to-address-list address-list=whatsapp_list address-list-timeout=1d chain=prerouting comment="WhatsApp Routing" content=.whatsapp. dst-address-list=!LOCAL src-address-list=LOCAL
```

# STEP 3 — STATIC WHATSAPP DOMAIN LIST
📘 Fungsi:
<p>Sebagai backup static apabila RAW tidak mendeteksi seluruh trafik.</p>

```shell
/ip firewall address-list
add address=whatsapp.com list=whatsapp_list
add address=v.whatsapp.com list=whatsapp_list
add address=account.whatsapp.com list=whatsapp_list
add address=chat.whatsapp.com list=whatsapp_list
add address=faq.whatsapp.com list=whatsapp_list
add address=web.whatsapp.com list=whatsapp_list
add address=www.whatsapp.com list=whatsapp_list
add address=api.whatsapp.com list=whatsapp_list
add address=autodiscover.whatsapp.com list=whatsapp_list
add address=b.whatsapp.com list=whatsapp_list
add address=blog.whatsapp.com list=whatsapp_list
add address=business.whatsapp.com list=whatsapp_list
add address=wa.me list=whatsapp_list
```
# STEP 4 — CREATE ROUTING TABLE
📘 Fungsi:
<p>Membuat tabel routing terpisah untuk trafik WhatsApp.</p>

```shell
/routing table add name="to_ISP1" fib comment="r_whatsapp"
```

# STEP 5 — ADD ROUTE
⚠️ Catatan: Ganti 0.0.0.0 dengan gateway ISP pilihan Anda.

```shell
/ip route add check-gateway=ping distance=1 gateway="0.0.0.0" routing-table="to_ISP1" comment="r_whatsapp"
```

# STEP 6 — MANGLE ROUTING
📘 Fungsi:
- Menandai trafik WhatsApp
- Memaksa trafik lewat gateway tertentu
- Mengurangi gangguan pada trafik umum

```shell
/ip firewall mangle add action=mark-routing chain=prerouting src-address-list=LOCAL dst-address-list="whatsapp_list" new-routing-mark="to_ISP1" comment="WhatsApp Routing" passthrough=no place-before=*0
```

# 🤝 Contribution
Pull Request terbuka untuk:
- Penambahan domain Meta
- Optimasi performa
- Script failover
- Dokumentasi tambahan

# 📜 License
MIT License — Free to use & modify.

🌟 Support
Jika script ini membantu:
<p>⭐ Star repository ini</p>
<p>🍴 Fork untuk pengembangan</p>
<p>🛠️ Kontribusi sangat terbuka</p>

<p align="center">
  <b>Made with ❤️ for MikroTik Engineers & Network Administrators</b>
</p>