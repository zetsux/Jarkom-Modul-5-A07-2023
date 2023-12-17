# Jarkom-Modul-5-A07-2023

**Praktikum Jaringan Komputer Modul 5 Tahun 2023**

## Author

| Nama                  | NRP        | Github                      |
| --------------------- | ---------- | --------------------------- |
| Dimas Fadilah Akbar   | 5025211010 | https://github.com/dimss113 |
| Kevin Nathanael Halim | 5025211140 | https://github.com/zetsux   |

# Laporan Resmi

## Daftar Isi

- [Laporan Resmi](#laporan-resmi)
  - [Daftar Isi](#daftar-isi)
  - [Task A (Topologi)](#topologi)
  - [Task B (VLSM)](#vlsm)
  - [Task C (Routing)](#routing)
  - [Task D (DHCP)](#dhcp)
  - [Soal 1](#soal-1)
  - [Soal 2](#soal-2)
  - [Soal 3](#soal-3)
  - [Soal 4](#soal-4)
  - [Soal 5](#soal-5)
  - [Soal 6](#soal-6)
  - [Soal 7](#soal-7)
  - [Soal 8](#soal-8)
  - [Soal 9](#soal-9)
  - [Soal 10](#soal-10)

## Task A (Topologi)

> Tugas pertama, buatlah peta wilayah sesuai berikut ini:
>
> <img src="https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/e1957afc-dee3-4d1d-b2a5-854c0906d260">
>
> Keterangan: Richter adalah DNS Server, Revolte adalah DHCP Server, Sein dan Stark adalah Web Server, Jumlah Host pada SchwerMountain adalah 64, Jumlah Host pada LaubHills adalah 255, Jumlah Host pada TurkRegion adalah 1022, Jumlah Host pada GrobeForest adalah 512

## Task B (VLSM)

> Untuk menghitung rute-rute yang diperlukan, gunakan perhitungan dengan metode VLSM. Buat juga pohonnya, dan lingkari subnet yang dilewati.

## Task C (Routing)

> Kemudian buatlah rute sesuai dengan pembagian IP yang kalian lakukan.

## Task D (DHCP)

> Tugas berikutnya adalah memberikan ip pada subnet SchwerMountain, LaubHills, TurkRegion, dan GrobeForest menggunakan bantuan DHCP.

## Soal 1

> Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Aura menggunakan iptables, tetapi tidak ingin menggunakan MASQUERADE.

## Soal 2

> Kalian diminta untuk melakukan drop semua TCP dan UDP kecuali port 8080 pada TCP.

## Soal 3

> Kepala Suku North Area meminta kalian untuk membatasi DHCP dan DNS Server hanya dapat dilakukan ping oleh maksimal 3 device secara bersamaan, selebihnya akan di drop.

## Soal 4

> Lakukan pembatasan sehingga koneksi SSH pada Web Server hanya dapat dilakukan oleh masyarakat yang berada pada GrobeForest.

### Scripts

- Range IP dari subnet warga GrobeForest adalah 192.172.4.1 - 192.172.7.254
- Buat sebuah file .sh di kedua web server (Sein & Stark), sebut saja `iptables.sh` yang menyimpan script untuk menambahkan iptables rules sebagai berikut,

  ```sh
  iptables -A INPUT -p tcp --dport 22 -m iprange --src-range 192.172.4.1-192.172.7.254  -j ACCEPT
  iptables -A OUTPUT -p tcp --sport 22 -m iprange --dst-range 192.172.4.1-192.172.7.254 -j ACCEPT
  iptables -A INPUT -p tcp --dport 22 -j DROP
  iptables -A OUTPUT -p tcp --sport 22 -j DROP
  ```

  #### Penjelasan

  - Yang dilakukan oleh baris ke-3 dan ke-4 adalah melakukan DROP terhadap semua request INPUT yang diarahkan ke port 22 (TCP) dan OUTPUT yang berasal dari port 22 (TCP), yaitu port default SSH
  - Kemudian ditambahkan rules pada baris ke-1 dan ke-2 untuk menerima request serupa hanya bila range ip tujuan merupakan bagian dari subnet warga GrobeForest untuk INPUT dan range ip asal merupakan bagian dari subnet warga GrobeForest untuk OUTPUT

### Bukti

- Setelah melakukan run pada script di atas, maka dapat dilakukan pembuktian dengan melakukan SSH dari GrobeForest ke web server sebagai berikut,

  - Stark (192.172.0.10) :
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/f951425a-d223-4cc0-91e3-1fa4a3bf3d8f)

  - Sein (192.172.4.2) :
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/e45315f8-82af-4a2a-ba79-208bb27a0f7d)

    Note : Didapatkan response "Connection refused" dikarenakan belum terdapat service ssh yang di-enable. Untuk sekedar testing apakah ssh berhasil atau tidak sebenarnya cukup sekian, tetapi bila ingin melakukan SSH yang sebenarnya maka dapat dilakukan install terhadap `openssh-server` dan diikuti `service ssh start`.

- Bila dilakukan dari warga dari subnet client lain, maka akan sebagai berikut

  - Stark (192.172.0.10) :
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/9c2fa319-2e51-436a-b62b-ee82debf1164)

  - Sein (192.172.4.2) :
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/4ae18b6b-085c-426b-a435-6fb25e4f211f)

    Note : Keluaran bukan kosong melainkan proses terus berjalan tanpa henti hingga di-terminate karena server ssh tidak dapat diraih.

## Soal 5

> Selain itu, akses menuju WebServer hanya diperbolehkan saat jam kerja yaitu Senin-Jumat pada pukul 08.00-16.00.

### Scripts

Pada `iptables.sh` di kedua web server (Sein & Stark), tambahkan script untuk menambahkan iptables rules di atas script untuk no. 4 sebagai berikut,

```sh
iptables -A INPUT -m time --weekdays Sat,Sun -j DROP
iptables -A INPUT -p all -m time --timestart 16:00 --timestop 23:59:59 -j DROP
iptables -A INPUT -p all -m time --timestart 00:00 --timestop 08:00 -j DROP
```

#### Penjelasan

- Baris ke-2 dan ke-3 melakukan DROP terhadap semua request INPUT di luar dari range waktu yang diinginkan (08:00 - 16:00), yakni sebelum jam 8 pagi (00:00 - 08:00) dan sesudah jam 4 sore (16:00 - 23:59:59)
- Baris ke-1 melakukan DROP terhadap semua request INPUT di luar hari kerja (Senin - Jumat), yakni pada hari Sabtu dan Minggu

### Bukti

- Sebelum itu perlu dipastikan waktu terkini dengan command `date`, hasilnya sebagai berikut,
  <br>
  ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/031e2d5d-a473-49ac-ab07-95b5bf7ae900)

- Karena waktu masih masuk di hari kerja dan jam kerja, maka waktu harus diatur terlebih dahulu ke hari lain. Untuk membuktikan di hari weekend maka waktu diset ke hari Minggu pukul 10:00 menggunakan command `date -s "17 dec 2023 10:00"`. Bila dilakukan tes dengan ping pada Sein diperoleh hasil sebagai berikut,

  - Sebelum ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/97e46ccf-46a0-4865-b044-be12b252c91a)

  - Setelah ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/f086ea65-92c3-4bf3-ba34-501921177b53)

- Untuk pembuktian jam kerja, maka waktu diatur ke hari kerja dengan jam di luar jam kerja dengan command `date -s "18 dec 2023 20:00"`. Bila dilakukan tes dengan ping pada Stark diperoleh hasil sebagai berikut,

  - Sebelum ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/d77478c1-5f31-43f4-985e-6a2d513e1e22)

  - Setelah ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/d7ff8315-b329-4145-a3db-6bfe58ba38ce)

## Soal 6

> Lalu, karena ternyata terdapat beberapa waktu di mana network administrator dari WebServer tidak bisa stand by, sehingga perlu ditambahkan rule bahwa akses pada hari Senin - Kamis pada jam 12.00 - 13.00 dilarang (istirahat maksi cuy) dan akses di hari Jumat pada jam 11.00 - 13.00 juga dilarang (maklum, Jumatan rek).

### Scripts

Pada `iptables.sh` di kedua web server (Sein & Stark), tambahkan script untuk menambahkan iptables rules di atas script untuk no. 5 sebagai berikut,

```sh
iptables -A INPUT -m time --timestart 12:00 --timestop 13:00 --weekdays Mon,Tue,Wed,Thu -j DROP
iptables -A INPUT -m time --timestart 11:00 --timestop 13:00 --weekdays Fri -j DROP
```

#### Penjelasan

- Baris ke-1 melakukan DROP terhadap semua request INPUT di hari Senin - Kamis pada jam makan siang (12:00 - 13:00)
- Baris ke-2 melakukan DROP terhadap semua request INPUT di hari Jumat pada jam jumatan (11:00 - 13:00)

### Bukti

- Untuk pembuktian di jam makan siang, maka waktu diset ke hari Senin pukul 12:30 menggunakan command `date -s "18 dec 2023 12:30"`. Bila dilakukan tes dengan ping pada Sein diperoleh hasil sebagai berikut,

  - Sebelum ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/647619e3-e415-4f6b-9af6-0b623eb9f8bc)

  - Setelah ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/ae9d8922-5c3a-4c3c-b893-b7b52158894b)

- Untuk pembuktian di jam jumatan, maka waktu diatur ke hari Jumat pada jam 12:30 dengan command `date -s "15 dec 2023 12:30"`. Bila dilakukan tes dengan ping pada Stark diperoleh hasil sebagai berikut,

  - Sebelum ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/b747a8cf-f236-4cb4-bab2-7a1078c51cfe)

  - Setelah ditambahkan iptables rules,
    <br>
    ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/39833116-406e-40d5-b7f9-381bf02ed85b)

## Soal 7

> Karena terdapat 2 WebServer, kalian diminta agar setiap client yang mengakses Sein dengan Port 80 akan didistribusikan secara bergantian pada Sein dan Stark secara berurutan dan request dari client yang mengakses Stark dengan port 443 akan didistribusikan secara bergantian pada Sein dan Stark secara berurutan.

### Scripts

Pada `iptables.sh` di router-router yang berkaitan dengan client (Heiter & Himmel), tambahkan script untuk menambahkan iptables rules sebagai berikut,

```sh
iptables -A PREROUTING -t nat -p tcp --dport 80 -d 192.172.4.2 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.172.0.10:80
iptables -A PREROUTING -t nat -p tcp --dport 443 -d 192.172.0.10 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.172.4.2:443
```

#### Penjelasan

- Baris ke-1 melakukan pengalihan request yang diarahkan ke IP Sein (192.172.4.2) pada port 80 agar secara bergantian diarahkan kesana dan juga ke Stark (192.172.0.10) pada port yang sama
- Baris ke-2 melakukan pengalihan request yang diarahkan ke IP Stark (192.172.0.10) pada port 443 agar secara bergantian diarahkan kesana dan juga ke Sein (192.172.4.2) pada port yang sama

### Bukti

- Sebelum pembuktian lakukan instalasi `netcat` pada semua web server dan client
- Untuk pembuktian di port 80, kita bisa jalankan script `while true; do nc -l -p 80 -c 'echo "stark"'; done` pada Stark dan script `while true; do nc -l -p 80 -c 'echo "sein"'; done` pada Sein. Bila dilakukan tes dengan `nc 192.172.4.2 80` pada client (sebagai contoh digunakan LaubHills, TurkRegion, dan SchwerMountain) dan akan diperoleh hasil sebagai berikut,

  - LaubHills :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184432465788534806/image.png?ex=658bf3b1&is=65797eb1&hm=96ec768396104ecceefd5b04644bafbb19b6e831ca22461910d69025da544ec5&)

  - TurkRegion :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184432578531446805/image.png?ex=658bf3cc&is=65797ecc&hm=1eddbada3b19f1afc6dcc86e89425155999e44a4f055ab5e6cf729b6ecfe2112&)

  - SchwerMountain :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184432735692009492/image.png?ex=658bf3f1&is=65797ef1&hm=ea4864821e380171656cbff8ff926ef6f4865f8ca6c0a11c3da75e17ca766c93&)

- Untuk pembuktian di port 443, kita bisa jalankan script `while true; do nc -l -p 443 -c 'echo "stark"'; done` pada Stark dan script `while true; do nc -l -p 443 -c 'echo "sein"'; done` pada Sein. Bila dilakukan tes dengan `nc 192.172.0.10 443` pada client (sebagai contoh digunakan LaubHills dan TurkRegion) dan akan diperoleh hasil sebagai berikut,

  - LaubHills :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184433440276357120/image.png?ex=658bf499&is=65797f99&hm=f5348b7a3f4ff334cd7daa382e00ecc017d13f36d02e6bd6322ea17060bcd48d&)

  - TurkRegion :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184433244192657408/image.png?ex=658bf46b&is=65797f6b&hm=176c18e694760cfa1f9b2d6bb6182251da85481d105794b4b6e89fc562ed36e5&)

  - SchwerMountain :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1184433537198325790/image.png?ex=658bf4b0&is=65797fb0&hm=b183133c3c427752645ba9186232813159eb918b1bec9dcc5dadfc0fb21333d0&)

## Soal 8

> Karena berbeda koalisi politik, maka subnet dengan masyarakat yang berada pada Revolte dilarang keras mengakses WebServer hingga masa pencoblosan pemilu kepala suku 2024 berakhir. Masa pemilu (hingga pemungutan dan penghitungan suara selesai) kepala suku bersamaan dengan masa pemilu Presiden dan Wakil Presiden Indonesia 2024.

### Scripts

Pada `iptables.sh` di kedua web server (Sein & Stark), tambahkan script untuk menambahkan iptables rules di atas script untuk no. 6 sebagai berikut,

```sh
iptables -A INPUT -m time --datestart "2023-10-19T00:00:00" --datestop "2024-6-28T00:00:00" -m iprange --src-range 192.172.0.21-192.172.0.24 -j DROP
```

#### Penjelasan

- Melakukan DROP terhadap semua request INPUT selama masa pemilu yakni diawali dari tanggal 19 November 2023 pukul 00:00 hingga tanggal 28 Juni 2024 pukul 00:00 (tanggal selesai perhitungan suara putaran kedua) untuk IP yang adalah bagian dari subnet Revolte yakni di antara 192.172.0.21 - 192.172.0.24

### Bukti

- Untuk pembuktian, maka waktu diset ke hari Jumat, 15 Desember 2023 pada pukul 10:00 menggunakan command `date -s "15 dec 2023 10:00" Bila dilakukan tes dengan ping dari Revolte pada Stark & Sein diperoleh hasil sebagai berikut,

  - Stark (192.172.0.10) :

    - Sebelum ditambahkan iptables rules,
      <br>
      ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/c9d3e8ef-efb7-4199-b884-dd8a356f872c)

    - Setelah ditambahkan iptables rules,
      <br>
      ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/1b81cd0d-3e28-4c9d-b02a-0bbff7dcb2f3)

  - Sein (192.172.4.2) :

    - Sebelum ditambahkan iptables rules,
      <br>
      ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/53aec99f-024c-4d29-bb71-4b800e7580f5)

    - Setelah ditambahkan iptables rules,
      <br>
      ![image](https://github.com/zetsux/Jarkom-Modul-5-A07-2023/assets/108170234/28193515-be2a-46ae-a9a1-f610bed59fdb)

## Soal 9

> Sadar akan adanya potensial saling serang antar kubu politik, maka WebServer harus dapat secara otomatis memblokir alamat IP yang melakukan scanning port dalam jumlah banyak (maksimal 20 scan port) di dalam selang waktu 10 menit. (clue: test dengan nmap)

### Scripts

Pada `iptables.sh` di kedua web server (Sein & Stark), tambahkan script untuk menambahkan iptables rules di antara script no. 4 dan 5 sebagai berikut,

```sh
iptables -N PORTSCAN
iptables -A INPUT -m recent --name portscan --update --seconds 600 --hitcount 20 -j DROP
iptables -A FORWARD -m recent --name portscan --update --seconds 600 --hitcount 20 -j DROP
iptables -A INPUT -m recent --name portscan --set -j ACCEPT
iptables -A FORWARD -m recent --name portscan --set -j ACCEPT
```

#### Penjelasan

- Melakukan DROP terhadap IP yang melakukan portscan / request INPUT maupun FORWARD lebih dari 20x hit dalam waktu 10 menit

### Bukti

- Untuk pembuktian, sebenarnya dapat dilakukan dengan `nmap`. Akan tetapi, akan lebih cepat bila menggunakan ping seperti biasa dan dilihat apakah setelah 20x ping masih dapat berjalan atau tidak. Contohnya dilakukan dari GrobeForest sebagai berikut,

  - Stark (192.172.0.10) :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1185250208058179594/image.png?ex=658eed46&is=657c7846&hm=8c79f2e7de39c70ccc8dc0664dd78d3872c5f275a8a67eed209332135fbc34c4&)

  - Sein (192.172.4.2) :
    <br>
    ![image](https://cdn.discordapp.com/attachments/1174671276728651806/1185250531552268360/image.png?ex=658eed93&is=657c7893&hm=669e512c4bae1bf1f7e323e5e55af370e571660ad65c8d482f8b0262deece42e&)

## Soal 10

> Karena kepala suku ingin tau paket apa saja yang di-drop, maka di setiap node server dan router ditambahkan logging paket yang di-drop dengan standard syslog level.

### Scripts

Pada `iptables.sh` di semua router dan server, tambahkan script untuk menambahkan iptables rules di atas script untuk no. 6 sebagai berikut,

```sh
service rsyslog restart
iptables -A INPUT -j LOG --log-prefix "Dropped: " --log-level debug -m limit --limit 1/second
```

#### Penjelasan

- Baris ke-2 melakukan logging setiap kali terdapat input dengan prefix Dropped yang menggunakan log level debug dengan limit 1 per detik
- Baris ke-1 menyalakan service rsyslog yang diperlukan untuk proses logging pada `/var/log/syslog`
