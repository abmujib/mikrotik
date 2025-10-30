**Buat profil khusus untuk malam hari dengan nama MALAM, misalnya dikasih limitasi kecil 10M/5M**
- Buat jadwal untuk malam hari lewat menu System > Scheduler.
- Name: Client Speed Malam
- Start Date: (terisi otomatis, biarkan)
- Start Time: 18:15:00
- Interval: 1d 00:00:00
- On Event:
```shell
/ppp secret set profile="MALAM" [find name="pppoe@pelanggan"]
:delay 2s;
/ppp active remove [find name="pppoe@pelanggan"]
```
*Silakan sesuaikan nama secret client dengan rumah pelanggan yang mau diatur jadwalnya.*
- Apply – OK.


**Buat jadwal lagi di System > Scheduler, untuk pagi harinya yang berfungsi untuk menormalkan kembali ke profil aslinya.**
- Name: Client Speed Normal
- Start Date: (terisi otomatis, biarkan)
- Start Time: 00:00:00
- Interval: 1d 00:00:00
- On Event:
```shell
/ppp secret set profile="profile_10M" [find name="pppoe@pelanggan"]
:delay 2s;
/ppp active remove [find name="pppoe@pelanggan"]
```
*Sesuaikan nama secret dan nama profil PPP.*
- Apply – OK.
