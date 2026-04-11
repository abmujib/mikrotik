<h1>Script LB PCC 2 ISP RouterOS V7 (new)</h1>
Berikut adalah skrip lengkap untuk konfigurasi Load Balancing metode PCC (Per Connection Classifier) dengan fitur Recursive Routing (Failover) khusus untuk MikroTik RouterOS v7.

1. Membuat Routing Table (Khusus v7)
Di RouterOS v7, routing mark harus dideklarasikan terlebih dahulu di tabel routing (FIB).
```shell
/routing table
add disabled=no fib name=to_ISP1
add disabled=no fib name=to_ISP2
```

2. Konfigurasi NAT (Masquerade)
Memastikan trafik dari subnet LOCAL bisa keluar melalui kedua ISP.
```shell
/ip firewall nat
add chain=srcnat out-interface=ether1 src-address-list=LOCAL action=masquerade comment="NAT ISP 1"
add chain=srcnat out-interface=ether2 src-address-list=LOCAL action=masquerade comment="NAT ISP 2"
```

3. Konfigurasi Firewall Mangle (PCC)
Bagian ini mengatur pembagian beban (load balance) 50:50 dan memastikan koneksi yang masuk dari ISP 1 akan keluar kembali lewat ISP 1 (begitu juga sebaliknya).
```shell
/ip firewall mangle
add chain=prerouting dst-address-list=LOCAL action=accept comment="Bypass Local Traffic"

add chain=prerouting in-interface=ether1 connection-mark=no-mark action=mark-connection new-connection-mark=conn_ISP1 passthrough=yes comment="Mark Conn Inbound ISP 1"
add chain=prerouting in-interface=ether2 connection-mark=no-mark action=mark-connection new-connection-mark=conn_ISP2 passthrough=yes comment="Mark Conn Inbound ISP 2"

add chain=prerouting src-address-list=LOCAL dst-address-type=!local connection-mark=no-mark per-connection-classifier=both-addresses:2/0 action=mark-connection new-connection-mark=conn_ISP1 passthrough=yes comment="PCC 1"
add chain=prerouting src-address-list=LOCAL dst-address-type=!local connection-mark=no-mark per-connection-classifier=both-addresses:2/1 action=mark-connection new-connection-mark=conn_ISP2 passthrough=yes comment="PCC 2"

add chain=prerouting connection-mark=conn_ISP1 src-address-list=LOCAL action=mark-routing new-routing-mark=to_ISP1 passthrough=no comment="Mark Route LAN to ISP 1"
add chain=prerouting connection-mark=conn_ISP2 src-address-list=LOCAL action=mark-routing new-routing-mark=to_ISP2 passthrough=no comment="Mark Route LAN to ISP 2"

add chain=output connection-mark=conn_ISP1 action=mark-routing new-routing-mark=to_ISP1 passthrough=no comment="Mark Route Output to ISP 1"
add chain=output connection-mark=conn_ISP2 action=mark-routing new-routing-mark=to_ISP2 passthrough=no comment="Mark Route Output to ISP 2"
```

4. Konfigurasi Recursive Routing (Failover)
Kita menggunakan IP DNS publik sebagai host pancingan (target pantau). ISP 1 dipantau melalui 1.0.0.1 sedangkan ISP 2 dipantau melalui 1.0.0.2
Jika salah satu ISP putus di tengah jalan (bukan kabel fisik putus, tapi akses internetnya mati), router akan mendeteksinya dari gagalnya PING ke IP DNS tersebut dan langsung mengalihkan jalur.
```shell
/ip route
add dst-address=1.0.0.1 gateway=192.168.148.1 scope=10 comment="Host Route ISP 1"
add dst-address=1.0.0.2 gateway=192.168.4.1 scope=10 comment="Host Route ISP 2"

add dst-address=0.0.0.0/0 gateway=1.0.0.1 routing-table=to_ISP1 check-gateway=ping target-scope=11 comment="PCC Route ISP 1"
add dst-address=0.0.0.0/0 gateway=1.0.0.2 routing-table=to_ISP2 check-gateway=ping target-scope=11 comment="PCC Route ISP 2"

add dst-address=0.0.0.0/0 gateway=1.0.0.1 routing-table=main check-gateway=ping distance=1 target-scope=11 comment="Main Failover Route 1"
add dst-address=0.0.0.0/0 gateway=1.0.0.2 routing-table=main check-gateway=ping distance=2 target-scope=11 comment="Main Failover Route 2"
```
