<h1>abmujib/mikrotik-failover-otomatis</h1>
Skrip MikroTik untuk mengatur failover otomatis antara 2 atau lebih sumber Internet

## Langkah #1: 
* Menambahkan IP Address untuk interface yang dijadikan sumber internet. (Sesuaikan IP address dan interface)
	```shell
	/ip address add address=192.168.1.2/24 interface=ether1-ISP1
	/
	/ip address add address=192.168.2.2/24 interface=ether2-ISP2
	/
	```

## Langkah #2: 
* Menambahkan rute default dengan jarak (distance)
	```shell
	/ip route add gateway=192.168.1.1 distance=1 check-gateway=ping
	/
	/ip route add gateway=192.168.2.1 distance=2
	/
	```

## Langkah #3: 
* Menambahkan Netwatch untuk pemantauan failover
	```shell
	/tool netwatch add host=192.168.1.1 interval=1m timeout=10s up-script="/ip route set [find gateway=192.168.1.1] distance=1; /ip route set [find gateway=192.168.2.1] distance=2" down-script="/ip route set [find gateway=192.168.1.1] distance=2; /ip route set [find gateway=192.168.2.1] distance=1"
	```

## Langkah #4: 
* Mengaktifkan logging untuk event Netwatch (opsional)
	```shell
	/system logging add topics=netwatch action=memory
	```

## Penjelasan skrip:

* Menetapkan Alamat IP untuk Antarmuka:
  - Menambahkan alamat IP 192.168.1.2/24 pada antarmuka ether1-ISP1.
  - Menambahkan alamat IP 192.168.2.2/24 pada antarmuka ether2-ISP2.

* Menambahkan Rute Default dengan Jarak (Distance):
  - Menambahkan rute default dengan gateway 192.168.1.1 (ISP1) dengan jarak 1 (prioritas utama) dan mengaktifkan pemeriksaan gateway (check-gateway=ping).
  - Menambahkan rute default dengan gateway 192.168.2.1 (ISP2) dengan jarak 2 (prioritas kedua).

* Menambahkan Netwatch untuk Pemantauan Failover:
  - Menambahkan Netwatch untuk memeriksa host 192.168.1.1 setiap 1 menit (interval=1m) dengan waktu tunggu 10 detik (timeout=10s).
  - up-script: Jika host 192.168.1.1 kembali aktif, skrip ini akan mengatur jarak rute ISP1 menjadi 1 dan ISP2 menjadi 2.
  - down-script: Jika host 192.168.1.1 tidak dapat dijangkau, skrip ini akan mengatur jarak rute ISP1 menjadi 2 dan ISP2 menjadi 1.

* Mengaktifkan Logging untuk Event Netwatch (Opsional):
  - Menambahkan logging untuk topik netwatch ke dalam log memori, sehingga setiap perubahan status akan dicatat.

Skrip ini memastikan bahwa jika koneksi ISP1 gagal, maka rute akan secara otomatis dialihkan ke ISP2, dan akan kembali ke ISP1 ketika koneksi sudah pulih.
