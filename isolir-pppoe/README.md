# ISOLIR PPPoE
Skrip MikroTik untuk Isolir Pelanggan PPPoE

## Langkah #1: 
* Buat IP Pool baru khusus untuk client PPPoE yang akan diisolir.
	```shell
	/ip pool add name=POOL_ISOLIR ranges=21.3.20.2-21.3.20.254
	```

## Langkah #2: 
* Buat Profile PPP baru khusus untuk isolir.
	```shell
	/ppp profile add name=PROFIL_ISOLIR local-address=21.3.20.1 remote-address=POOL_ISOLIR address-list=ISOLIR_LIST
	```

## Langkah #3: 
* Aktifkan Web Proxy dan Atur Port.
	```shell
	/ip proxy set enabled=yes port=8080
	```
	```shell
	/ip proxy access add comment="Blok PPPoE Isolir - ERTENET" src-address=21.3.20.0/24 dst-address=!21.3.20.1 action=deny redirect-to=21.3.20.1:8080
	```

## Langkah #4: 
* Buat notifikasi ke perangkat client (HP / PC) bahwa akses internet telah diisolir.
	```shell
	/ip firewall nat add comment="Notif Redirect PPPOE Isolir - ERTENET" chain=dstnat protocol=tcp dst-port=80,443 src-address-list=ISOLIR_LIST action=redirect to-ports=8080
	```

## Langkah #5: 
* Buat rule untuk memblokir semua akses internet khusus ke PPPoE yang terisolir.
	```shell
	/ip firewall filter add comment="Blok Internet ke PPPoE Isolir - ERTENET" chain=forward protocol=tcp dst-port=!8080 src-address-list=ISOLIR_LIST action=drop
	```

## Langkah #6: 
* Reset dan Hapus File error.html bawaan Mikrotik.
	```shell
	/ip proxy reset-html
	y
	/file remove webproxy/error.html
	```
	
## Langkah #7:
* Download file error.html
	[Download error.html]([https://raw.githubusercontent.com/abmujib/mikrotik-isolir-pppoe/main/error.html](https://raw.githubusercontent.com/abmujib/mikrotik/main/isolir-pppoe/error.html))

Catatan: Silakan sesuaikan lokasi dan nama folder yang ditandai di atas dengan tipe penyimpanan Mikrotik Anda, misalnya menjadi /file remove flash/webproxy/error.html atau bisa /file remove disk/webproxy/error.html
