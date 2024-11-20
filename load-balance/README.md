Script konfigurasi Mikrotik sistem load balancing mode Per-Connection Classifier (LB PCC) dengan failover recursive gateway. Pastikan kecepatan internet yang digunakan sama, atau akan menyamaratakan pada kecepatan terendah.

Contoh Kasus:
* Jumlah ISP = 2
* Versi ROS = v6
* Nama Interface ISP-1: ether1-ISP1
* Nama Interface ISP-2: ether2-ISP2
* IP Gateway ISP-1: 192.168.100.1
* IP Gateway ISP-2: 192.168.100.2

Hubungkan sumber internet ke Mikrotik dengan DHCP-Client. Anda bisa melakukannya secara manual atau gunakan kode berikut. Pastikan fungsi "Add Default Route" dimatikan.
```shell
/ip dhcp-client add interface="ether1-ISP1" add-default-route=no disabled=no
/ip dhcp-client add interface="ether2-ISP2" add-default-route=no disabled=no
```

Aktifkan DNS Server pada Mikrotik.
```shell
/ip dns set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
```

Buat Global Masquerade untuk mempermudah dan mempercepat konfigurasi.
```shell
/ip firewall nat add action=masquerade chain=srcnat comment="Global NAT Masquerade - ABMUJIB" place-before=*0
```

Buat Address List LOCAL menggunakan Private IP berikut. Jika anda tidak menggunakan network IP yang satu segmen dengan daftar di bawah ini, silakan tambahkan sendiri secara manual.
```shell
/ip firewall address-list add address=10.0.0.0/8 list=LOCAL
/ip firewall address-list add address=172.16.0.0/12 list=LOCAL
/ip firewall address-list add address=192.168.0.0/16 list=LOCAL
```

Buat Route untuk masing-masing gateway ISP. Fitur failover recursive akan menggunakan IP DNS 1.0.0.1 ~ 1.0.0.4 sebagai cek gateway.
```shell
/ip route add distance=1 dst-address=1.0.0.1 gateway=192.168.100.1 comment="LB-PCC-RES"
/ip route add distance=1 dst-address=1.0.0.2 gateway=192.168.200.1 comment="LB-PCC-RES"
/ip route add check-gateway=ping distance=1 gateway=1.0.0.1 target-scope=30 comment="LB-PCC-RES"
/ip route add check-gateway=ping distance=2 gateway=1.0.0.2 target-scope=30 comment="LB-PCC-RES"
/ip route add check-gateway=ping distance=1 gateway=1.0.0.1 routing-mark="via-ISP1" target-scope=30 comment="LB-PCC-RES"
/ip route add check-gateway=ping distance=1 gateway=1.0.0.2 routing-mark="via-ISP2" target-scope=30 comment="LB-PCC-RES"
```

Konfigurasi Load Balance PCC.
```shell
/ip firewall mangle add action=accept chain=prerouting dst-address-list=LOCAL src-address-list=LOCAL comment="Accept All LOCAL IP - ABMUJIB"
/ip firewall mangle add action=accept chain=postrouting dst-address-list=LOCAL src-address-list=LOCAL
/ip firewall mangle add action=accept chain=forward dst-address-list=LOCAL src-address-list=LOCAL
/ip firewall mangle add action=accept chain=input dst-address-list=LOCAL src-address-list=LOCAL
/ip firewall mangle add action=accept chain=output dst-address-list=LOCAL src-address-list=LOCAL
/ip firewall mangle add action=mark-connection chain=input in-interface="ether1-ISP1" new-connection-mark="via-ether1-ISP1" passthrough=yes comment="Load Balance PCC - ABMUJIB"
/ip firewall mangle add action=mark-connection chain=input in-interface="ether2-ISP2" new-connection-mark="via-ether2-ISP2" passthrough=yes
/ip firewall mangle add action=mark-routing chain=output connection-mark="via-ether1-ISP1" new-routing-mark="via-ISP1" passthrough=yes
/ip firewall mangle add action=mark-routing chain=output connection-mark="via-ether2-ISP2" new-routing-mark="via-ISP2" passthrough=yes
/ip firewall mangle add action=mark-connection chain=prerouting dst-address-type=!local new-connection-mark="via-ether1-ISP1" passthrough=yes per-connection-classifier=both-addresses-and-ports:2/0 dst-address-list=!LOCAL src-address-list=LOCAL
/ip firewall mangle add action=mark-connection chain=prerouting dst-address-type=!local new-connection-mark="via-ether2-ISP2" passthrough=yes per-connection-classifier=both-addresses-and-ports:2/1 dst-address-list=!LOCAL src-address-list=LOCAL
/ip firewall mangle add action=mark-routing chain=prerouting connection-mark="via-ether1-ISP1" new-routing-mark="via-ISP1" passthrough=yes dst-address-list=!LOCAL src-address-list=LOCAL
/ip firewall mangle add action=mark-routing chain=prerouting connection-mark="via-ether2-ISP2" new-routing-mark="via-ISP2" passthrough=yes dst-address-list=!LOCAL src-address-list=LOCAL
```
