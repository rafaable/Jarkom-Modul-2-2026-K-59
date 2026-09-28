# Laporan Resmi Praktikum Komdat-Jarkom Modul 2

> **Kelompok K-59**
> * Salsabila Rafa Syafira (5027251059)
> * Fiorellin Ilona (5027251082)

---

## Daftar Isi

* [Soal 1](#soal-1)
* [Soal 2](#soal-2)
* [Soal 3](#soal-3)
* [Soal 4](#soal-4)
* [Soal 5](#soal-5)
* [Soal 6](#soal-6)
* [Soal 7](#soal-7)
* [Soal 8](#soal-8)
* [Soal 9](#soal-9)
* [Soal 10](#soal-10)
* [Soal 11](#soal-11)
* [Soal 12](#soal-12)
* [Soal 13](#soal-13)
* [Soal 14](#soal-14)
* [Soal 15](#soal-15)
* [Soal 16](#soal-16)
* [Soal 17](#soal-17)
* [Soal 18](#soal-18)
* [Soal 19](#soal-19)
* [Soal 20](#soal-20)

---
Prefix : 10.93.x.x
[Kredensial GNS3](http://10.4.89.250/)

# Pembahasan

## Soal 1
### Perintah soal:  
1. Router rootkit harus tersambung ke lima switch. Di topologi, rootkit terhubung ke Switch1, Switch4, Switch5, Switch6, dan Switch7, ditambah satu kabel lagi ke NAT  
2. Setiap node diberi alamat IP sesuai switch tempat dia tersambung, prefix kelompokmu: 10.93.x.x.  
3. Setiap node non-router diberi default gateway, yaitu IP rootkit di jaringan tempat node itu berada.  
4. Seluruh entitas tetap harus diberi IP  

Switch2 dan Switch3 tidak langsung tersambung ke rootkit. Mereka menggantung di bawah Switch1. Artinya prab, tedd, obladi, desmond, oblada, dan molly berada di satu kompleks yang sama (satu subnet), karena switch yang disambung ke switch lain hanya memperpanjang jalan, tidak membuat kompleks baru. Hanya router yang bisa memisahkan kompleks  
<br><img width="1307" height="843" alt="image" src="https://github.com/user-attachments/assets/c049f2f9-6f78-4568-8f39-e1fec0ee2f05" />
<br>  

### Pemetaan perangkat & port
| Perangkat A | Port A | Perangkat B | Port B | Keterangan Perangkat B |
| --- | --- | --- | --- | --- |
| NAT1 | nat0 | rootkit | eth0 | Jalur ke internet (WAN) |
| rootkit | eth1 | Switch6 | e0 | Gerbang ke sayap kiri (klien pengamat) |
| rootkit | eth2 | Switch4 | e0 | Gerbang ke abbey (reverse proxy) |
| rootkit | eth3 | Switch1 | e0 | Gerbang ke zona server (DNS + web) |
| rootkit | eth4 | Switch5 | e0 | Gerbang ke penny (reverse proxy) |
| rootkit | eth5 | Switch7 | e0 | Gerbang ke sayap kanan (klien eksekutor) |
| Switch6 | e1 | alpha | e0 | Klien sayap kiri |
| Switch6 | e2 | beta | e0 | Klien sayap kiri |
| Switch6 | e3 | gamma | e0 | Klien sayap kiri |
| Switch7 | e1 | delta | e0 | Klien sayap kanan |
| Switch7 | e2 | epsilon | e0 | Klien sayap kanan |
| Switch4 | e1 | abbey | eth0 | Reverse proxy Nginx (ke area core) |
| Switch5 | e1 | penny | eth0 | Reverse proxy Apache (ke area vault) |
| Switch1 | e1 | Switch2 | e0 | Penyambung ke zona DNS |
| Switch1 | e2 | Switch3 | e0 | Penyambung ke zona web |
| Switch2 | e1 | prab | eth0 | DNS master (ns1) |
| Switch2 | e2 | tedd | eth0 | DNS slave (ns2) |
| Switch3 | e1 | obladi | eth0 | Web statis Apache (area vault) |
| Switch3 | e2 | desmond | eth0 | Web statis Apache (area vault) |
| Switch3 | e3 | oblada | eth0 | Web dinamis Nginx + PHP-FPM (area core) |
| Switch3 | e4 | molly | eth0 | Web dinamis Nginx + PHP-FPM (area core) |

### Tabel subnet rootkit:  
| Subnet | Pintu rootkit | Switch | Node dan IP | Gateway |
| --- | --- | --- | --- | --- |
| Internet | `eth0` (DHCP) | NAT1 | – | – |
| `10.93.1.0/24` | `eth1` = `10.93.1.1` | Switch6 | alpha `.2`, beta `.3`, gamma `.4` | `10.93.1.1` |
| `10.93.2.0/24` | `eth2` = `10.93.2.1` | Switch4 | abbey `.2` | `10.93.2.1` |
| `10.93.3.0/24` | `eth3` = `10.93.3.1` | Switch1 (+2, +3) | prab `.2`, tedd `.3`, obladi `.4`, desmond `.5`, oblada `.6`, molly `.7` | `10.93.3.1` |
| `10.93.4.0/24` | `eth4` = `10.93.4.1` | Switch5 | penny `.2` | `10.93.4.1` |
| `10.93.5.0/24` | `eth5` = `10.93.5.1` | Switch7 | delta `.2`, epsilon `.3` | `10.93.5.1` |

### Konfigurasi rootkit  
```bash
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 10.93.1.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 10.93.2.1
    netmask 255.255.255.0

auto eth3
iface eth3 inet static
    address 10.93.3.1
    netmask 255.255.255.0

auto eth4
iface eth4 inet static
    address 10.93.4.1
    netmask 255.255.255.0

auto eth5
iface eth5 inet static
    address 10.93.5.1
    netmask 255.255.255.0
```

### Konfigurasi node lain
```
auto eth0
iface eth0 inet static
    address 10.93.a.b
    netmask 255.255.255.0
    gateway 10.93.c.d
```
| Node | address | gateway |
| --- | --- | --- |
| alpha | `10.93.1.2` | `10.93.1.1` |
| beta | `10.93.1.3` | `10.93.1.1` |
| gamma | `10.93.1.4` | `10.93.1.1` |
| abbey | `10.93.2.2` | `10.93.2.1` |
| prab | `10.93.3.2` | `10.93.3.1` |
| tedd | `10.93.3.3` | `10.93.3.1` |
| obladi | `10.93.3.4` | `10.93.3.1` |
| desmond | `10.93.3.5` | `10.93.3.1` |
| oblada | `10.93.3.6` | `10.93.3.1` |
| molly | `10.93.3.7` | `10.93.3.1` |
| penny | `10.93.4.2` | `10.93.4.1` |
| delta | `10.93.5.2` | `10.93.5.1` |
| epsilon | `10.93.5.3` | `10.93.5.1` |

### Check check check!
Di console rootkit  
```
ip a
ip route
```
Cek semua node sekaligus dari rootkit
```
for ip in 10.93.1.2 10.93.1.3 10.93.1.4 10.93.2.2 10.93.3.2 10.93.3.3 10.93.3.4 10.93.3.5 10.93.3.6 10.93.3.7 10.93.4.2 10.93.5.2 10.93.5.3; do
  ping -c 1 -W 1 $ip > /dev/null && echo "$ip OK" || echo "$ip GAGAL"
done
```
Nanti harus keluar
```
10.93.1.2 OK
10.93.1.3 OK
10.93.1.4 OK
10.93.2.2 OK
10.93.3.2 OK
10.93.3.3 OK
10.93.3.4 OK
10.93.3.5 OK
10.93.3.6 OK
10.93.3.7 OK
10.93.4.2 OK
10.93.5.2 OK
10.93.5.3 OK
```
Cek gateway tiap-tiap node dengan:
```
ip route
```
Harus sesuai dengan IP Gateway masing masing, beres.
---

## Soal 2
### Perintah soal
1. WAN adalah kabel ke "dunia luar", yaitu eth0, yang tersambung ke NAT1. Antarmuka WAN di rootkit harus aktif, yangmana dianggap aman ketika eth0 sudah dapat IP 192.168.122.x dari DHCP.
2. Konfigurasi NAT di rootkit supaya semua host internal (10.93.x.x) bisa menjangkau internet menggunakan IP address. Artinya pengujiannya cukup ping ke IP publik seperti 8.8.8.8, belum ke nama domain seperti google.com

Alamat 10.93.x.x itu alamat privat, semacam nomor rumah di dalam kompleks tertutup. Kalau alpha mengirim surat ke luar dengan alamat pengirim 10.93.1.2, balasannya tidak akan pernah bisa kembali karena kantor pos di luar tidak tahu alamat itu ada di mana. Solusinya bisa melalui IP forwarding, atau NAT (MASQUERADE) -> kirim atas nama gateway, rootkit mengganti alamat pengirimnya dengan alamat rootkit sendiri (192.168.122.x), yang dikenal dunia luar. Rootkit juga mencatat "surat ini sebenarnya punya alpha"

### Langkah pengerjaan
1. Tambahkan perintah NAT ke network config rootkit
   ```
   auto eth0
   iface eth0 inet dhcp
       up echo 1 > /proc/sys/net/ipv4/ip_forward
       up iptables -t nat -A POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE
   ```
2. Simpan juga sebagai script di /root
   ```
   cat > /root/soal2.sh <<'EOF'
   #!/bin/bash
   echo 1 > /proc/sys/net/ipv4/ip_forward
   iptables -t nat -A POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE
   EOF
   chmod +x /root/soal2.sh
   ```
3. Jangan lupa stop, lalu start biar config barunya keimplement!
4. Di rootkit, cek dua hal:
   ```
   cat /proc/sys/net/ipv4/ip_forward
   iptables -t nat -L POSTROUTING -n -v
   ```
   Hasil pertama harus 1. Hasil kedua harus menampilkan satu baris MASQUERADE dengan sumber 10.93.0.0/16 dan keluar lewat eth0. Di beberapa node dari subnet berbeda (misalnya alpha, abbey, prab, penny, delta), ping ke IP publik:
   ```
   ping -c 3 8.8.8.8
   ```
   Kalau ada reply, nomor 2 selesai. Ingat? ujinya pakai IP. `ping google.com` masih belum karena resolver belum diatur.
   > Notes : Smpe sini semper error gabisa ping, wkwkwk
   > Troubleshootnya:
   > 1. Step 1 hapus dulu dari config, lalu restart
   > 2. Jalankan `bash /root/soal2.sh`
   > 3. Jalankan `iptables -t nat -L POSTROUTING -n -v`
   > 4. ping -c 3 8.8.8.8
<br><img width="519" height="99" alt="image" src="https://github.com/user-attachments/assets/50c72ebf-5cbb-4276-aaf0-50cd30a19eec" /><br>
Great, sekarang coba juga `ping -c 3 8.8.8.8` dari alpha
<br><img width="574" height="188" alt="image" src="https://github.com/user-attachments/assets/151df8d2-4722-4efa-88f5-fcfd59e32740" /><br>  

Sisa satu pekerjaan: membuat NAT ini aktif otomatis saat rootkit di-restart tanpa merusak proses boot seperti tadi
<br>
Langkah 1: Tulis ulang script /root/soal2.sh  
Di console rootkit:
```
cat > /root/soal2.sh <<'EOF'
#!/bin/bash
# Pastikan program seperti iptables bisa ditemukan saat boot
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Izinkan rootkit meneruskan paket
sysctl -w net.ipv4.ip_forward=1 || echo 1 > /proc/sys/net/ipv4/ip_forward

# Tambah aturan NAT hanya kalau belum ada, supaya tidak dobel
iptables -t nat -C POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE 2>/dev/null || \
iptables -t nat -A POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE
EOF
chmod +x /root/soal2.sh
```
Tiga pengamannya:
* export PATH=... memastikan iptables dan sysctl tetap ketemu walaupun proses boot memakai daftar folder yang terbatas.  
* sysctl ... || echo ... artinya kalau cara pertama ditolak, coba cara kedua.
* iptables -C ... || iptables -A ...   
artinya cek dulu apakah aturannya sudah ada, dan tambahkan hanya kalau belum. Jadi kalau script jalan dua kali, aturannya tidak dobel  

Langkah 2: Panggil script dari network config  
Klik kanan rootkit → Edit network configuration, lalu ubah bagian eth0 saja menjadi:  
```
auto eth0
iface eth0 inet dhcp
    up /bin/bash /root/soal2.sh || true
```
Langkah 3: Stop lalu Start rootkit, tunggu sekitar 20 detik, lalu cek di rootkit:
```
ip a | grep inet
cat /proc/sys/net/ipv4/ip_forward
iptables -t nat -L POSTROUTING -n -v
```
Lalu di alpha:
```
ping -c 3 8.8.8.8
```
Hasil yang diharapkan:  
* Semua interface punya IP.  
* ip_forward bernilai 1.  
* Ada satu baris MASQUERADE.  
* Alpha bisa ping 8.8.8.8.
  
<br><img width="704" height="368" alt="image" src="https://github.com/user-attachments/assets/54451ed9-2418-45ec-a996-8f8ad8bb3aed" /><br>
<br><img width="571" height="183" alt="image" src="https://github.com/user-attachments/assets/4eeae2f1-46ac-46ea-abc7-a0f16404a5a8" /><br>
Aman yaps.




## Soal 3
### Perintah soal
1. Routing internal via rootkit berfungsi. Semua node harus bisa saling berkomunikasi lintas subnet lewat rootkit. Misalnya alpha (10.93.1.x) bisa menjangkau prab (10.93.3.x), penny (10.93.4.x), dan delta (10.93.5.x).
2. Tambahkan resolver 192.168.122.1 di /etc/resolv.conf pada setiap node non-router, dan resolver ini harus terpasang saat interface-nya aktif (otomatis tiap node menyala). Soal juga menegaskan: kalau sudah pakai resolver itu, tidak perlu menambah resolver Google (8.8.8.8).

### Langkah pengerjaan
1. Untuk 13 node (alpha, beta, gamma, delta, epsilon, abbey, penny, prab, tedd, obladi, desmond, oblada, molly): klik kanan → Edit network configuration, lalu tambahkan satu baris up di bawah gateway
   ```
   up echo nameserver 192.168.122.1 > /etc/resolv.conf || true
   ```
2. Restart semua node non-router
   Stop lalu Start ke-13 node itu. Rootkit dan NAT1 biarkan menyala.
3. Langkah 3: Simpan script di /root (untuk memenuhi aturan soal). Di setiap node non-router:
   ```
   echo 'echo nameserver 192.168.122.1 > /etc/resolv.conf' > /root/soal3.sh
   ```
4. Verifikasi routing lintas subnet
   Dari alpha, ping satu perwakilan setiap subnet lain:
   ```
   for ip in 10.93.2.2 10.93.3.2 10.93.3.7 10.93.4.2 10.93.5.2; do ping -c 1 -W 1 $ip > /dev/null && echo "$ip OK" || echo "$ip GAGAL" done
   ``` 
   Untuk bukti bahwa jalurnya memang lewat rootkit, jalankan di alpha:  
   ```
   traceroute 10.93.3.2
   ```
   Lompatan pertama harus 10.93.1.1 (rootkit), lalu lompatan kedua 10.93.3.2 (prab)  
5. Cek resolver
   ```
   cat /etc/resolv.conf
   ping -c 3 google.com
   ```
<br><img width="834" height="507" alt="image" src="https://github.com/user-attachments/assets/500e0950-0161-4334-8027-cd1de22b5c5d" /><br>  
All done!

---

## Soal 4
### Perintah soal
Penataan resolver setelah DNS . Domain kelompokmu: k59.com. DNS tidak membedakan huruf besar dan kecil, jadi K59.com dan k59.com sama saja. Kita pakai huruf kecil supaya rapi.  
1. Konfigurasi di prab (ns1 / Master)  
   Buat zona `k59.com` yang authoritative, dengan SOA menunjuk ke `prab.k59.com`.
2. Tambahkan NS record untuk:
   - `prab.k59.com`
   - `tedd.k59.com`
3. Tambahkan A record:
   - `prab.k59.com` → `10.93.3.2`
   - `tedd.k59.com` → `10.93.3.3`
   - `k59.com` → `10.93.4.2` (IP penny)
4. Aktifkan notify dan allow-transfer ke tedd  
5. Set forwarders ke `192.168.122.1`  
6. Konfigurasi di tedd (ns2 / Slave)
   - Tarik zona `k59.com` dari prab sebagai slave  
   - Pastikan tedd menjawab query zona `k59.com` secara authoritative  
7. Konfigurasi resolver di semua node non-router
    Ubah urutan resolver menjadi:
    - `10.93.3.2` → prab
    - `10.93.3.3` → tedd
    - `192.168.122.1` → DNS forwarder
8. Verifikasi
    Lakukan query DNS ke:
    - `k59.com`
    - `prab.k59.com`
    - `tedd.k59.com`
    Pastikan query dijawab dengan benar oleh prab atau tedd dan response bersifat authoritative.
9. Target hasil
    | Hostname | Record | IP |
    |---|---|---|
    | `k59.com` | A | `10.93.4.2` |
    | `prab.k59.com` | A | `10.93.3.2` |
    | `tedd.k59.com` | A | `10.93.3.3` |
10. DNS Server  
    | Server | Role | IP |
    |---|---|---|
    | `prab` | Master / ns1 | `10.93.3.2` |
    | `tedd` | Slave / ns2 | `10.93.3.3` |
    
DNS ini bisa dibayangkan seperti buku telepon khusus untuk jaringan kita. Node prab menjadi pemilik buku telepon resmi untuk domain k59.com, sehingga disebut authoritative. Di dalam zona tersebut ada beberapa record: SOA yang berisi informasi utama dan nomor versi zona, NS yang menentukan server DNS resmi yaitu prab dan tedd, serta A record yang menghubungkan nama domain dengan alamat IP. k59.com sendiri disebut apex, yaitu nama domain utama tanpa tambahan nama di depannya. Jika dicek menggunakan dig, jawaban authoritative biasanya ditandai dengan flag aa.  
  
prab berperan sebagai master, sedangkan tedd sebagai slave yang memiliki salinan zona dari prab. allow-transfer menentukan bahwa tedd boleh mengambil salinan zona, sementara notify memberi tahu tedd ketika zona di prab mengalami perubahan. Karena memiliki salinan resmi, tedd juga bisa memberikan jawaban authoritative. Untuk domain yang tidak diketahui, seperti google.com, prab meneruskan pertanyaan ke 192.168.122.1 melalui forwarder. Sementara itu, semua node menggunakan urutan resolver prab → tedd → 192.168.122.1, sehingga jika prab tidak bisa diakses, pertanyaan dapat dilanjutkan ke tedd, lalu ke DNS cadangan.

### Pengerjaan soal
1. Buat script di console prab
   ```
   nano /root/soal4.sh
   ```
   Isi scriptnya
   ```
    #!/bin/bash
    apt-get update
    apt-get install -y bind9 dnsutils
    ln -sf /etc/init.d/named /etc/init.d/bind9
    
    # Opsi global: forwarders dan izin query
    cat > /etc/bind/named.conf.options <<'EOF'
    options {
        directory "/var/cache/bind";
        forwarders { 192.168.122.1; };
        dnssec-validation no;
        allow-query { any; };
        allow-recursion { any; };
        auth-nxdomain no;
        listen-on-v6 { any; };
    };
    EOF
    
    # Deklarasi zona master
    cat > /etc/bind/named.conf.local <<'EOF'
    zone "k59.com" {
        type master;
        notify yes;
        also-notify { 10.93.3.3; };
        allow-transfer { 10.93.3.3; };
        file "/etc/bind/k59/k59.com";
    };
    EOF
    
    # Isi zona (buku telepon)
    mkdir -p /etc/bind/k59
    cat > /etc/bind/k59/k59.com <<'EOF'
    $TTL    604800
    @       IN      SOA     prab.k59.com. root.k59.com. (
                            2026092801 ; Serial
                            604800     ; Refresh
                            86400      ; Retry
                            2419200    ; Expire
                            604800 )   ; Negative Cache TTL
    ;
    @       IN      NS      prab.k59.com.
    @       IN      NS      tedd.k59.com.
    @       IN      A       10.93.4.2   ; apex -> penny
    prab    IN      A       10.93.3.2
    tedd    IN      A       10.93.3.3
    EOF
    
    named-checkconf && named-checkzone k59.com /etc/bind/k59/k59.com
    service named restart
   ```
   Simpan lalu jalankan
   ```
   bash /root/soal4.sh
   ```
    Beberapa hal di script ini yang tidak ada di modul, tapi sengaja kutambahkan:
    * `allow-recursion { any; };` Secara bawaan, BIND hanya mau mencarikan jawaban domain luar (seperti `google.com`) untuk penanya dari subnet yang sama dengan dirinya. prab ada di `10.93.3.x`, jadi alpha (`10.93.1.x`) akan ditolak (REFUSED) kalau tanya google.com. Baris ini membuka izin untuk semua subnet
    * Titik di akhir nama (`prab.k59.com.`). Titik itu artinya "ini nama lengkap, jangan ditambah apa-apa lagi". Tanpa titik, BIND akan membacanya sebagai `prab.k59.com.k59.com`. Ini persis materi Modul DNS bagian 3 poin 2 (Penggunaan Titik)
    * Serial `2026092801` mengikuti format tanggal hari ini + nomor urut (Modul DNS bagian 3 poin 1). Setiap kali zona diubah, angka ini harus dinaikkan, misalnya menjadi `2026092802`. Kalau tidak, tedd tidak akan menyalin versi barunya
    * `named-checkconf` dan `named-checkzone` memeriksa typo sebelum BIND dinyalakan. Kalau ada kesalahan, pesannya muncul di sini
    <br>
    Hasil yang diharapkan di akhir: zone k59.com/IN: loaded serial 2026092801 dan OK
2. Di console tedd(ns2, slave), `nano /root/soal4.sh`, lalu tempel:
   ```
    #!/bin/bash
    apt-get update
    apt-get install -y bind9 dnsutils
    ln -sf /etc/init.d/named /etc/init.d/bind9
    
    cat > /etc/bind/named.conf.options <<'EOF'
    options {
        directory "/var/cache/bind";
        forwarders { 192.168.122.1; };
        dnssec-validation no;
        allow-query { any; };
        allow-recursion { any; };
        auth-nxdomain no;
        listen-on-v6 { any; };
    };
    EOF
    
    cat > /etc/bind/named.conf.local <<'EOF'
    zone "k59.com" {
        type slave;
        masters { 10.93.3.2; };
        file "/var/lib/bind/k59.com";
    };
    EOF
    
    named-checkconf
    service named restart
    ```
    Lalu jalankan `bash /root/soal4.sh`
   tedd juga diberi forwarders dan allow-recursion, supaya kalau prab mati, tedd tetap bisa menjawab pertanyaan domain luar
3. Pastikan tedd sudah menyalin zona, di console tedd
   ```
   ls -l /var/lib/bind/
   ```
   Total 0 bjirr, bentar troubleshoot. Harusnya ada file k59.com. Kalau belum ada, restart BIND di prab supaya dia kirim notify lagi (service named restart di prab), tunggu beberapa detik, terus cek lagi.
   Cara cek apakah bind-nya running
   ```
   root@tedd:~# service named status
   ```
   > bind is running.
   Diagnosis di prab
   ```
    cat /etc/resolv.conf
    dpkg -l | grep bind9
    which named
    ls -l /etc/init.d/ | grep -E "named|bind"
   ```
   <br><img width="775" height="223" alt="image" src="https://github.com/user-attachments/assets/9394a529-3df5-4815-8f09-cf96bb8efe7d" /><br>
   Yang terinstall di prab hanya bind9-dnsutils, bind9-host, dan bind9-libs. Itu cuma alat untuk bertanya ke DNS (dig, host) dan pustaka pendukungnya, kemungkinan besar sudah bawaan image. Paket bind9 itu sendiri, yaitu server DNS-nya, tidak pernah terinstall. Makanya which named kosong dan /etc/init.d/ tidak punya file named maupun bind9. Ibaratnya, prab punya telepon untuk menelepon kantor informasi, tapi kantor informasinya sendiri belum pernah dibangun
   <br>
   ```
   apt-get update
   DEBIAN_FRONTEND=noninteractive apt-get install -y -o Dpkg::Options::="--force-confnew" bind9
   ```
   Pastikan paketnya sudah terpasang
   ```
   ls -l /etc/init.d/ | grep -E "named|bind"
   ```
   Kalau /usr/sbin/named dan file named sudah muncul, jalankan ulang script-nya:
   ```
   bash /root/soal4.sh
   ```
   <br><img width="996" height="791" alt="image" src="https://github.com/user-attachments/assets/7fe36f5e-d38d-4e93-9a8d-5441ecef1e22" /><br>
   Oh yeah, so far so good yeah? OK cek prab dulu
   ```
   dig @127.0.0.1 k59.com SOA
   ```
   Picu di tedd
   ```
   service named restart
   sleep 3
   ls -l /var/lib/bind/
   dig @10.93.3.3 k59.com SOA
   ```
   <br><img width="740" height="406" alt="image" src="https://github.com/user-attachments/assets/673f495f-9e1f-4fb8-9375-3a4c261dc87b" /><br>
   <br><img width="743" height="526" alt="image" src="https://github.com/user-attachments/assets/32e00c85-cafb-4181-993d-0dd757642159" /><br>

   Dari screenshot itu:
   * status: NOERROR artinya pertanyaan dijawab tanpa masalah.
   * flags: qr aa rd ra, dan yang terpenting aa: prab menjawab sebagai pemilik resmi zona k59.com (authoritative).
   * SOA prab.k59.com. root.k59.com. 2026092801: sampul buku teleponnya sesuai soal, dengan pemilik utama prab dan serial 2026092801.
4. Di network config prab dan tedd,  pasang autostart BIND di prab dan tedd lalu tambahkan baris ini di bawah baris up lainnya:
   ```
   up /usr/sbin/service named start || true
   ```
5. Ubah urutan resolver di ke-13 node non-router  
   ```
    up echo nameserver 10.93.3.2 > /etc/resolv.conf || true
    up echo nameserver 10.93.3.3 >> /etc/resolv.conf || true
    up echo nameserver 192.168.122.1 >> /etc/resolv.conf || true
   ```
   Semua node masih bertanya ke 192.168.122.1, dan kantor informasi itu tidak kenal k59.com. Jadi kalau alpha menjalankan ping k59.com sekarang, hasilnya gagal.
   Step ini memberi tahu setiap node ke mana harus bertanya:
   * prab dulu, karena dia pemilik resmi k59.com. Untuk domain luar seperti google.com, prab meneruskannya lewat forwarders.
   * tedd kalau prab tidak menjawab, karena dia punya salinan yang sama.
   * 192.168.122.1 sebagai cadangan terakhir kalau dua-duanya mati, supaya internet (misalnya untuk apt-get) tetap bisa jalan  
     
   Lalu terapkan langsung di console masing-masing supaya tidak perlu restart:
   ```
   printf 'nameserver 10.93.3.2\nnameserver 10.93.3.3\nnameserver 192.168.122.1\n' > /etc/resolv.conf
   ```
   Verifikasi dari klien, misal di alpha
   ```
    apk add bind-tools
    dig k59.com
    dig prab.k59.com
    dig tedd.k59.com
    dig k59.com NS
    dig @10.93.3.3 k59.com
    dig google.com
   ```
   Tujuan setiap verifikasi klien
   Soal meminta: *"Verifikasi bahwa query ke domain apex maupun hostname di dalam zona dijawab dengan benar oleh prab atau tedd."* Tiap perintah membuktikan satu bagian:

    | Perintah | Yang dibuktikan |
    | --- | --- |
    | `apk add bind-tools` | Memasang `dig` di alpha. Alpine tidak punya `dig` bawaan. |
    | `dig k59.com` | **Apex** mengarah ke penny (`10.93.4.2`) sesuai soal. |
    | `dig prab.k59.com` | **Hostname di dalam zona** (A record prab) benar. |
    | `dig tedd.k59.com` | A record tedd benar. |
    | `dig k59.com NS` | Kedua nameserver resmi (prab dan tedd) sudah terdaftar. |
    | `dig @10.93.3.3 k59.com` | tedd juga menjawab dengan benar dan authoritative **dari sudut pandang klien**. Tadi kamu mengetesnya dari tedd sendiri; sekarang dari node lain lewat jaringan. |
    | `dig google.com` | **Forwarders** di prab berfungsi, jadi walaupun resolver pertama sekarang prab, internet tetap jalan. |
   <br>
   Di setiap hasil, perhatikan dua baris kunci:
    - **`flags:` harus ada `aa`** (untuk domain `k59.com`). Ini bukti jawabannya datang dari sumber resmi, bukan dari cache atau tebakan.
    - **`SERVER:` harus `10.93.3.2`** (atau `10.93.3.3` untuk yang pakai `@`). Ini bukti yang menjawab benar-benar prab atau tedd, bukan `192.168.122.1`. Artinya Step 3 sudah bekerja.
    
    Khusus `dig google.com`, flag `aa` **tidak akan muncul**, dan itu benar. prab bukan pemilik `google.com`; dia hanya menanyakannya ke pihak lain lalu meneruskan jawabannya
   <br><img width="551" height="775" alt="image" src="https://github.com/user-attachments/assets/8774df1d-0864-4f28-b595-7598bf576059" /><br>
   <br><img width="467" height="587" alt="image" src="https://github.com/user-attachments/assets/bb17b1e7-421e-4666-9628-8eef2b792f1b" /><br>
   <br><img width="444" height="600" alt="image" src="https://github.com/user-attachments/assets/bf4309eb-c896-4897-b847-2c592c7795e5" /><br>
   Semuanya lolos. Ini bukti lengkap untuk nomor 4:
    | Pengujian | Hasil | Keterangan |
    | --- | --- | --- |
    | `dig k59.com` | `10.93.4.2` | Apex mengarah ke penny |
    | `dig prab.k59.com` | `10.93.3.2` | A record prab benar |
    | `dig tedd.k59.com` | `10.93.3.3` | A record tedd benar |
    | `dig k59.com NS` | `prab.k59.com.`, `tedd.k59.com.` | Dua nameserver resmi terdaftar |
    | `dig @10.93.3.3 k59.com` | `10.93.4.2`, dijawab tedd | Slave menjawab authoritative dari sisi klien |
    | `dig google.com` | 6 IP Google | Forwarders di prab jalan |  
   <br><img width="497" height="257" alt="image" src="https://github.com/user-attachments/assets/82a1313c-2471-49e8-8817-491b956cfb45" /><br>
   Setelah prab di-restart, alpha tetap mendapat jawaban 10.93.4.2 dengan flag aa dan SERVER: 10.93.3.2. Artinya BIND di prab menyala sendiri saat boot, dan autostart-nya berhasil. Dengan ini, nomor 4 selesai sepenuhnya:
   * Zona k59.com authoritative di prab, dengan SOA, NS, dan A record sesuai soal.
   * Notify dan allow-transfer ke tedd, dan tedd menjawab authoritative dengan serial yang sama.
   * Forwarders ke 192.168.122.1 berfungsi.
   * Urutan resolver prab → tedd → 192.168.122.1 aktif.
   * BIND di prab dan tedd autostart, sekaligus mencicil kebutuhan nomor 20.
---

## Soal 5
### Perintah soal
1. Beri hostname ke semua node sesuai glosarium: rootkit, alpha, beta, gamma, delta, epsilon, prab, tedd, abbey, penny, obladi, desmond, oblada, molly.
2. Verifikasi bahwa setiap host mengenali hostname-nya secara system-wide, artinya seluruh sistem di node itu (bukan cuma tampilan prompt) tahu siapa dirinya.
3. Buat domain untuk setiap node dengan format namanode.k59.com (misalnya alpha.k59.com) yang mengarah ke IP-nya masing-masing. Pengecualian: prab dan tedd, karena A record mereka sudah dibuat di nomor 4<br>

Ada tiga "tempat nama" yang berbeda, dan soal ini menyentuh ketiganya<br>
1. Hostname (/etc/hostname): papan nama di depan rumah. Ini nama yang dipakai node untuk menyebut dirinya sendiri. Nama inilah yang muncul di prompt (root@prab), di log, dan ketika program bertanya "saya ini siapa?". Karena kamu sudah menamai node di GNS3, kemungkinan besar papan namanya sudah terpasang. Tugas kita memastikan nama itu juga tertulis permanen di file, bukan cuma tampil di layar.

2. /etc/hosts: buku catatan pribadi. Sebelum bertanya ke kantor informasi (DNS), setiap node selalu membuka buku catatan kecilnya sendiri dulu. Kalau nama dirinya tercatat di situ, node bisa mengenali namanya sendiri bahkan kalau DNS sedang mati. Inilah makna "system-wide": namanya dikenal oleh sistem, bukan bergantung pada pihak luar.

3. A record di DNS (prab): buku telepon kota. Ini supaya node lain bisa menemukan alpha dengan nama alpha.k59.com. Papan nama dan buku catatan pribadi hanya berguna untuk diri sendiri. Agar tetangga bisa mencari, nama itu harus terdaftar di buku telepon resmi

### Langkah pengerjaan
Tambahkan semua A record di prab. Tentukan dulu satu dari lima IP router untuk rootkit, dipilih 10.93.3.1, yaitu pintu yang menghadap zona server (prab dan tedd), karena itu IP rootkit yang paling "dekat" dengan layanan DNS.
1. Di prab, buat `nano /root/soal5.sh` dan tempel:
    ```
    #!/bin/bash
    cat > /etc/bind/k59/k59.com <<'EOF'
    $TTL    604800
    @       IN      SOA     prab.k59.com. root.k59.com. (
                            2026092802 ; Serial
                            604800     ; Refresh
                            86400      ; Retry
                            2419200    ; Expire
                            604800 )   ; Negative Cache TTL
    ;
    @       IN      NS      prab.k59.com.
    @       IN      NS      tedd.k59.com.
    @       IN      A       10.93.4.2   ; apex -> penny
    
    ; Penjaga direktori (nomor 4)
    prab    IN      A       10.93.3.2
    tedd    IN      A       10.93.3.3
    
    ; Domain tiap node (nomor 5)
    rootkit IN      A       10.93.3.1
    alpha   IN      A       10.93.1.2
    beta    IN      A       10.93.1.3
    gamma   IN      A       10.93.1.4
    abbey   IN      A       10.93.2.2
    obladi  IN      A       10.93.3.4
    desmond IN      A       10.93.3.5
    oblada  IN      A       10.93.3.6
    molly   IN      A       10.93.3.7
    penny   IN      A       10.93.4.2
    delta   IN      A       10.93.5.2
    epsilon IN      A       10.93.5.3
    EOF
    
    named-checkzone k59.com /etc/bind/k59/k59.com
    service named restart
    ```
    Jalankan `bash /root/soal5.sh`

    <br><img width="507" height="79" alt="image" src="https://github.com/user-attachments/assets/bdea494d-0ff5-415f-baf3-72e65a5fcefe" /><br>
2. Pastikan tedd atau alpha ikut sinkron
   ```
   dig @10.93.3.3 k59.com SOA +short
   ```
   > prab.k59.com. root.k59.com. 2026092802 604800 86400 2419200 604800
3. Di setiap node, `nano /root/soal5-hostname.sh` (di prab, pakai nama file ini supaya tidak menimpa `soal5.sh` dari Langkah 1):

      ```sh
      #!/bin/sh
      NAME=alpha
      IP=10.93.1.2
      
      hostname $NAME
      echo $NAME > /etc/hostname
      grep -q "$NAME.k59.com" /etc/hosts || echo "$IP $NAME.k59.com $NAME" >> /etc/hosts
      ```

      Isi `NAME` dan `IP` sesuai node:
      
      | Node | NAME | IP |
      | --- | --- | --- |
      | rootkit | `rootkit` | `10.93.3.1` |
      | alpha | `alpha` | `10.93.1.2` |
      | beta | `beta` | `10.93.1.3` |
      | gamma | `gamma` | `10.93.1.4` |
      | abbey | `abbey` | `10.93.2.2` |
      | prab | `prab` | `10.93.3.2` |
      | tedd | `tedd` | `10.93.3.3` |
      | obladi | `obladi` | `10.93.3.4` |
      | desmond | `desmond` | `10.93.3.5` |
      | oblada | `oblada` | `10.93.3.6` |
      | molly | `molly` | `10.93.3.7` |
      | penny | `penny` | `10.93.4.2` |
      | delta | `delta` | `10.93.5.2` |
      | epsilon | `epsilon` | `10.93.5.3` |
   Jalankan di tiap node..
   ```
   sh /root/soal5-hostname.sh
   ```
4. Di network config setiap node, tambahkan satu baris up di bawah baris-baris up yang sudah ada:
   ```
    up /bin/sh /root/soal5-hostname.sh || true
   ```
   Khusus rootkit, taruh baris ini di bawah iface eth0 inet dhcp, setelah baris up /bin/bash /root/soal2.sh || true

5. Testing, misal gamma
   ```
    hostname
    cat /etc/hostname
    grep "$(hostname)" /etc/hosts
    ping -c 1 "$(hostname)"
   ```
   Cek semua domain sekaligus
   ```
   for n in rootkit alpha beta gamma delta epsilon prab tedd abbey penny obladi desmond oblada molly; do echo "$n.k59.com -> $(dig +short $n.k59.com)" done
   ```
   <br><img width="629" height="325" alt="image" src="https://github.com/user-attachments/assets/9a1f2704-57c6-477a-b95b-1bdfc1211405" /><br>
   Hasil loop dari abbey sempurna: ke-12 domain mengarah ke IP yang benar sesuai tabel. Karena abbey ada di subnet 10.93.2.x, ini juga sudah jadi bukti dari "klien kedua" selain alpha. Soal cat yang gagal: itu karena menjalankannya di abbey. File zona hanya ada di prab, sang pemilik buku telepon. abbey cuma "penanya", jadi dia memang tidak punya file itu. Jalankan di console prab:
   ```
   cat /etc/bind/k59/k59.com
   ```
   Ulangi loop yang sama di delta, supaya terbukti hasilnya konsisten dari subnet lain.
   Untuk satu atau dua contoh, tampilkan dig lengkap supaya flag aa dan SERVER: 10.93.3.2 terlihat
   <br><img width="530" height="513" alt="image" src="https://github.com/user-attachments/assets/a5c7deaa-347e-4b52-a014-402a7b8fb5cb" /><br>
   Bukti sebelumnya hanya menunjukkan "buku teleponnya bilang begitu"; bukti ini menunjukkan "teleponnya benar-benar tersambung ke orang yang dimaksud". Dari delta:
    ```
    ping -c 2 alpha.k59.com
    ping -c 2 molly.k59.com
    ping -c 2 rootkit.k59.com
    ```
   <br><img width="453" height="571" alt="image" src="https://github.com/user-attachments/assets/2a5e5726-570a-48ad-94c6-0624b252e2df" /><br>
   Ada reply artinya nama itu diterjemahkan ke IP yang benar, dan di alamat itu memang ada node yang menjawab.

---

## Soal 6
*Deskripsi dan pembahasan soal nomor 6. Tolong dilanjut yaa, fio*

Bonus: tedd juga punya catatan yang sama
Karena soal nomor 6 nanti meminta prab dan tedd identik, sekalian buktikan dari alpha:
sh
for n in rootkit alpha beta gamma delta epsilon abbey penny obladi desmond oblada molly; do
  echo "$n.k59.com -> $(dig @10.93.3.3 +short $n.k59.com)"
done
---

## Soal 7
*Deskripsi dan pembahasan soal nomor 7.*

---

## Soal 8
*Deskripsi dan pembahasan soal nomor 8.*

---

## Soal 9
*Deskripsi dan pembahasan soal nomor 9.*

---

## Soal 10
*Deskripsi dan pembahasan soal nomor 10.*

---

## Soal 11
*Deskripsi dan pembahasan soal nomor 11.*

---

## Soal 12
*Deskripsi dan pembahasan soal nomor 12.*

---

## Soal 13
*Deskripsi dan pembahasan soal nomor 13.*

---

## Soal 14
*Deskripsi dan pembahasan soal nomor 14.*

---

## Soal 15
*Deskripsi dan pembahasan soal nomor 15.*

---

## Soal 16
*Deskripsi dan pembahasan soal nomor 16.*

---

## Soal 17
*Deskripsi dan pembahasan soal nomor 17.*

---

## Soal 18
*Deskripsi dan pembahasan soal nomor 18.*

---

## Soal 19
*Deskripsi dan pembahasan soal nomor 19.*

---

## Soal 20
*Deskripsi dan pembahasan soal nomor 20.*
