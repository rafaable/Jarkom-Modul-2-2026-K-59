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
* [Soal 12 (Revisi)](#soal-12-revisi)
* [Soal 13](#soal-13)
* [Soal 14](#soal-14)
* [Soal 15](#soal-15)
* [Soal 15 (Revisi)](#soal-15-revisi)
* [Soal 16](#soal-16)
* [Soal 17](#soal-17)
* [Soal 17 (Revisi)](#soal-17-revisi)
* [Soal 18](#soal-18)
* [Soal 19](#soal-19)
* [Soal 20](#soal-20)
* [Revisi](#revisi)

---
Prefix : 10.93.x.x
[Kredensial GNS3](http://10.4.89.250/)

# Pembahasan

## Soal 1
### Perintah soal:  
1. Router rootkit harus tersambung ke lima switch. Di topologi, rootkit terhubung ke Switch1, Switch4, Switch5, Switch6, dan Switch7, ditambah satu kabel lagi ke NAT  
2. Setiap node diberi alamat IP sesuai switch tempat dia tersambung, prefix kelompok: 10.93.x.x.  
3. Setiap node non-router diberi default gateway, yaitu IP rootkit di jaringan tempat node itu berada.  
4. Seluruh entitas tetap harus diberi IP  

Switch2 dan Switch3 tidak langsung tersambung ke rootkit. Mereka menggantung di bawah Switch1. Artinya prab, tedd, obladi, desmond, oblada, dan molly berada di satu kompleks yang sama (satu subnet), karena switch yang disambung ke switch lain hanya memperpanjang jalan, tidak membuat kompleks baru. Hanya router yang bisa memisahkan kompleks  
<br><img width="1415" height="883" alt="image" src="https://github.com/user-attachments/assets/277bf20a-d399-4b1f-853e-dc51fc3e3400" /><br>  

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
1. X Tambahkan perintah NAT ke network config rootkit
   ```
   auto eth0
   iface eth0 inet dhcp
       up echo 1 > /proc/sys/net/ipv4/ip_forward
       up iptables -t nat -A POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE
   ```
2. X Simpan juga sebagai script di /root
   ```
   cat > /root/soal2.sh <<'EOF'
   #!/bin/bash
   echo 1 > /proc/sys/net/ipv4/ip_forward
   iptables -t nat -A POSTROUTING -s 10.93.0.0/16 -o eth0 -j MASQUERADE
   EOF
   chmod +x /root/soal2.sh
   ```
3. X Jangan lupa stop, lalu start biar config barunya keimplement!
4. X Di rootkit, cek dua hal:
   ```
   cat /proc/sys/net/ipv4/ip_forward
   iptables -t nat -L POSTROUTING -n -v
   ```
   Hasil pertama harus 1. Hasil kedua harus menampilkan satu baris MASQUERADE dengan sumber 10.93.0.0/16 dan keluar lewat eth0. Di beberapa node dari subnet berbeda (misalnya alpha, abbey, prab, penny, delta), ping ke IP publik:
   ```
   ping -c 3 8.8.8.8
   ```
   Kalau ada reply, nomor 2 selesai. Ingat? ujinya pakai IP. `ping google.com` masih belum karena resolver belum diatur
   
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
   for ip in 10.93.2.2 10.93.3.2 10.93.3.7 10.93.4.2 10.93.5.2; do ping -c 1 -W 1 $ip > /dev/null && echo "$ip OK" || echo "$ip GAGAL"; done
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
Penataan resolver setelah DNS . Domain kelompok: k59.com. DNS tidak membedakan huruf besar dan kecil, jadi K59.com dan k59.com sama saja. Kita pakai huruf kecil supaya rapi.  
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
    apt-get install -y bind9 dnsutils             # instal bind9 & alat buat uji DNS terus auto jawab yes
    ln -sf /etc/init.d/named /etc/init.d/bind9    # service named juga bisa dipanggil dengan nama bind9
    
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
    $TTL    43200
    @       IN      SOA     prab.k59.com. root.k59.com. (
                            2026092801 ; Serial
                            43200      ; Refresh
                            3600       ; Retry
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
   service named status
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
    $TTL    43200
    @       IN      SOA     prab.k59.com. root.k59.com. (
                            2026092802 ; Serial
                            43200      ; Refresh
                            3600       ; Retry
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
   bash /root/soal5-hostname.sh
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
   * `hostname` buat pembuktian sistem mengenali diri sendiri
   * `cat /etc/hostname` buat buktiin nama hostname udah disimpen di node bersangkutan
   * `grep "$(hostname)" /etc/hosts` nama hostname udah dipetakan belom IP nya
   * `ping` IP nya bisa dihubungi nggak
     
   Cek semua domain sekaligus
   ```
   for n in rootkit alpha beta gamma delta epsilon prab tedd abbey penny obladi desmond oblada molly; do echo "$n.k59.com -> $(dig +short $n.k59.com)"; done
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
**Perintah soal:**
- Pastikan *zone transfer* berjalan lancar dari `prab` (Master) ke `tedd` (Slave).
- Pastikan `tedd` telah menerima salinan zona terbaru dari `prab`.
- Nilai serial SOA di kedua DNS server harus sama persis karena keduanya tidak bisa dipisahkan dan saling melengkapi.

**Konsep & Pembahasan:**
 DNS *Master-Slave* bekerja dengan mekanisme replikasi zona. Ketika ada penambahan atau perubahan record pada server Master (`prab`), nomor serial SOA (*Start of Authority*) di file zona dinaikkan (misalnya dari `2026092801` menjadi `2026092802`).

Server Master akan mengirimkan sinyal *notify* ke IP Slave (`tedd`), lalu server Slave akan meminta proses *zone transfer* (AXFR/IXFR) untuk menyalin data zona terbaru. Dengan demikian, `tedd` dapat memberikan jawaban *authoritative* yang identik dengan `prab` kepada seluruh klien di dalam jaringan.

**Langkah pengerjaan:**

**1. Pastikan konfigurasi izin transfer pada Master (`prab`)**
   Di console `prab`, pastikan file `/etc/bind/named.conf.local` telah mendeklarasikan `notify yes;`, `also-notify`, dan `allow-transfer` ke IP `tedd` (`10.93.3.3`):
   ```bash
   cat > /etc/bind/named.conf.local <<'EOF'
   zone "k59.com" {
       type master;
       notify yes;
       also-notify { 10.93.3.3; };
       allow-transfer { 10.93.3.3; };
       file "/etc/bind/k59/k59.com";
   };
   EOF
```
**2. Pastikan konfigurasi zona Slave pada tedd**
Di console tedd, pastikan zona k59.com diatur sebagai type slave; yang mengarah ke Master (10.93.3.2):

```Bash
cat > /etc/bind/named.conf.local <<'EOF'
zone "k59.com" {
    type slave;
    masters { 10.93.3.2; };
    file "/var/lib/bind/k59.com";
};
EOF
```
**3. Pemicuan Zone Transfer**
Restart service BIND di kedua node agar proses replikasi dipicu:

Di prab: service named restart

Di tedd: service named restart

**4. Verifikasi Salinan Zona di tedd**
Cek apakah berkas zona dari prab sudah berhasil diterima dan disimpan oleh tedd:

```Bash
ls -l /var/lib/bind/
Hasil: Terdapat berkas k59.com yang menandakan salinan zona telah diterima dari Master.
```
**5. Pengujian Klien (dari alpha)**
Jalankan query SOA ke Master (prab) dan Slave (tedd) untuk memastikan kesamaan nilai serial:

```Bash
dig @10.93.3.2 k59.com SOA +short
dig @10.93.3.3 k59.com SOA +short
```
Hasil Output:

Plaintext
prab.k59.com. root.k59.com. 2026092802 604800 86400 2419200 604800
prab.k59.com. root.k59.com. 2026092802 604800 86400 2419200 604800
**Pengujian Resolusi Seluruh Hostname via tedd**
Lakukan loop testing dari klien alpha untuk memastikan tedd meresolusi seluruh A Record node:

```Bash
for n in rootkit alpha beta gamma delta epsilon abbey penny obladi desmond oblada molly; do
  echo "$n.k59.com -> $(dig @10.93.3.3 +short $n.k59.com)"
```
done
Hasil Output:

```bash
Plaintext
rootkit.k59.com -> 10.93.3.1
alpha.k59.com -> 10.93.1.2
beta.k59.com -> 10.93.1.3
gamma.k59.com -> 10.93.1.4
delta.k59.com -> 10.93.5.2
epsilon.k59.com -> 10.93.5.3
abbey.k59.com -> 10.93.2.2
penny.k59.com -> 10.93.4.2
obladi.k59.com -> 10.93.3.4
desmond.k59.com -> 10.93.3.5
oblada.k59.com -> 10.93.3.6
molly.k59.com -> 10.93.3.7
```
<img width="888" height="807" alt="image" src="https://github.com/user-attachments/assets/dfb6432f-aaef-4e27-a498-7dfb7da23959" />

---

## Soal 7
### Perintah soal
1. `abbey` dan `penny` sebagai gerbang utama, `obladi` dan `desmond` sebagai web statis, `oblada` dan `molly` sebagai web dinamis.
2. Tambahkan pada zona `k59.com` A record untuk:
   - `vault.k59.com` (mengarah ke IP `obladi` & `desmond`)
   - `core.k59.com` (mengarah ke IP `oblada` & `molly`)
3. Tetapkan CNAME:
   - `www.k59.com` → `penny.k59.com`
   - `static.k59.com` → `abbey.k59.com`
4. Verifikasi dari dua klien berbeda bahwa seluruh hostname tersebut ter-resolve ke tujuan yang benar dan konsisten.

### Konsep & Pembahasan
Pada soal ini, kita memperluas konfigurasi DNS (`k59.com`) di Master (`prab`) dengan menambahkan dua jenis record penting:
* **Multiple A Record (Round-Robin):** Satu nama domain (`vault` dan `core`) dihubungkan ke dua alamat IP sekaligus untuk mendistribusikan titik akses ke beberapa server.
* **CNAME (Canonical Name):** Membuat alias nama domain. Domain `www` diarahkan ke `penny`, dan `static` diarahkan ke `abbey`.

### Langkah pengerjaan
**1. Perbarui file zona master di `prab`**
   Edit atau timpa file zona `/etc/bind/k59/k59.com` dengan menaikkan nomor serialnya (misalnya dari `2026092802` menjadi `2026092803`) dan tambahkan record berikut di bagian bawah:
   ```bash
   cat > /etc/bind/k59/k59.com <<'EOF'
   $TTL    604800
   @       IN      SOA     prab.k59.com. root.k59.com. (
                           2026092803 ; Serial
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

   ; --- TAMBAHAN SOAL 7 ---
   vault   IN      A       10.93.3.4
   vault   IN      A       10.93.3.5
   core    IN      A       10.93.3.6
   core    IN      A       10.93.3.7
   www     IN      CNAME   penny
   static  IN      CNAME   abbey
   EOF

   named-checkzone k59.com /etc/bind/k59/k59.com
   rndc reload
   ```
  **2. Sinkronisasi Slave di tedd**
Pastikan BIND di tedd memuat perubahan zona terbaru:
```bash
pkill -9 named && /usr/sbin/named -u bind
```
**3. Verifikasi dari Dua Klien Berbeda (alpha dan delta)**
Jalankan perintah pengujian berikut di console alpha dan delta:

```Bash
dig +short vault.k59.com
dig +short core.k59.com
dig +short [www.k59.com](https://www.k59.com)
dig +short static.k59.com
```
<img width="1110" height="842" alt="image" src="https://github.com/user-attachments/assets/14f65239-7c27-4c95-aca6-6f8f60ff9363" />

<img width="1140" height="845" alt="image" src="https://github.com/user-attachments/assets/97b8a768-d98d-4e12-8939-3cf886b8adc5" />

<img width="1122" height="847" alt="image" src="https://github.com/user-attachments/assets/e1dda0a1-15d9-453b-9f27-0d929443f32f" />

  
---

## Soal 8
### Perintah Soal
1. Di `prab` (ns1) deklarasikan reverse zone untuk segmen jaringan tempat `abbey, penny,` area `vault`, dan area `core` berada.
2. Di `tedd` (ns2) tarik reverse zone tersebut sebagai slave.
3. Isi record PTR untuk keempat hostname tersebut agar pencarian balik (reverse lookup) IP address mengembalikan hostname yang benar.
4. Pastikan query reverse untuk alamat-alamat tersebut dijawab secara authoritative.

### Konsep & Pembahasan
Reverse DNS (PTR record) berfungsi untuk memetakan alamat IP kembali menjadi nama domain (kebalikan dari A record). Karena perangkat berada di subnet berbeda (`10.93.2.x, 10.93.3.x`, dan `10.93.4.x`), kita mendeklarasikan beberapa reverse zone (`in-addr.arpa`) di Master prab lalu mereplikasikannya ke Slave tedd.

### Langkah Pengerjaan
**1. Deklarasi Reverse Zone di Master (prab)**
Tambahkan konfigurasi zona PTR ke /etc/bind/named.conf.local:

```Bash
cat >> /etc/bind/named.conf.local <<'EOF'

zone "2.93.10.in-addr.arpa" {
    type master;
    notify yes;
    also-notify { 10.93.3.3; };
    allow-transfer { 10.93.3.3; };
    file "/etc/bind/k59/rev.2";
};

zone "3.93.10.in-addr.arpa" {
    type master;
    notify yes;
    also-notify { 10.93.3.3; };
    allow-transfer { 10.93.3.3; };
    file "/etc/bind/k59/rev.3";
};

zone "4.93.10.in-addr.arpa" {
    type master;
    notify yes;
    also-notify { 10.93.3.3; };
    allow-transfer { 10.93.3.3; };
    file "/etc/bind/k59/rev.4";
};
EOF
```
Buat file pemetaan PTR masing-masing subnet di /etc/bind/k59/:

File rev.2 (Subnet 10.93.2.x untuk abbey di .2):

```Bash
cat > /etc/bind/k59/rev.2 <<'EOF'
$TTL    604800
@       IN      SOA     prab.k59.com. root.k59.com. (2026092801 604800 86400 2419200 604800)
@       IN      NS      prab.k59.com.
@       IN      NS      tedd.k59.com.
2       IN      PTR     abbey.k59.com.
EOF
```
File rev.3 (Subnet 10.93.3.x untuk vault & core):

```Bash
cat > /etc/bind/k59/rev.3 <<'EOF'
$TTL    604800
@       IN      SOA     prab.k59.com. root.k59.com. (2026092801 604800 86400 2419200 604800)
@       IN      NS      prab.k59.com.
@       IN      NS      tedd.k59.com.
4       IN      PTR     obladi.k59.com.
5       IN      PTR     desmond.k59.com.
6       IN      PTR     oblada.k59.com.
7       IN      PTR     molly.k59.com.
EOF
```
File rev.4 (Subnet 10.93.4.x untuk penny di .2):

```Bash
cat > /etc/bind/k59/rev.4 <<'EOF'
$TTL    604800
@       IN      SOA     prab.k59.com. root.k59.com. (2026092801 604800 86400 2419200 604800)
@       IN      NS      prab.k59.com.
@       IN      NS      tedd.k59.com.
2       IN      PTR     penny.k59.com.
EOF
```
Simpan file, atur hak akses kepemilikan (chown -R bind:bind /etc/bind/k59), lalu jalankan rndc reload.

**2. Konfigurasi Slave Reverse Zone di tedd**
Tambahkan blok zona slave yang sama ke /etc/bind/named.conf.local di tedd lalu restart BIND.

**3. Verifikasi Reverse Lookup dari Klien (alpha)**
```Bash
dig -x 10.93.2.2 +short
dig -x 10.93.4.2 +short
dig -x 10.93.3.4 +short
dig -x 10.93.3.6 +short
```
Hasil yang diharapkan:
Masing-masing IP sukses mengembalikan hostname yang sesuai (abbey.k59.com., penny.k59.com., obladi.k59.com., oblada.k59.com.).

<img width="1142" height="845" alt="image" src="https://github.com/user-attachments/assets/383e3320-dd9a-4373-b682-d555b6ebbc5f" />


---

## Soal 9
### Perintah Soal
1. Jalankan layanan web statis pada hostname di node area `vault` (menggunakan Apache).
2. Buka folder direktori `/arsip/` dan aktifkan fitur autoindex (directory listing) pada konfigurasi Apache sehingga seluruh daftar file di dalamnya dapat ditelusuri langsung dari browser.
3. Akses pengujian wajib dilakukan melalui hostname, bukan IP address.

### Langkah pengerjaan
**1. Instalasi dan Konfigurasi Apache di Node Vault (obladi)**

```Bash
apt-get update
apt-get install -y apache2
```

**2. Buat direktori arsip dan isi beberapa file dummy**
```bash
mkdir -p /var/www/html/arsip
echo "Dokumen Rahasia Vault Obladi" > /var/www/html/arsip/rahasia1.txt
echo "Laporan Keuangan Vault" > /var/www/html/arsip/laporan.pdf
```

**3. Buat file konfigurasi untuk mengaktifkan autoindex**
```bash
cat > /etc/apache2/conf-available/arsip-index.conf <<'EOF'
<Directory /var/www/html/arsip>
    Options +Indexes
    AllowOverride All
    Require all granted
</Directory>
EOF

a2enconf arsip-index.conf
service apache2 restart
```

Verifikasi dari Klien (alpha)
Uji akses direktori arsip menggunakan hostname lewat curl:

```Bash
curl http://obladi.k59.com/arsip/
curl http://vault.k59.com/arsip/
```

Hasil: Menampilkan halaman HTML Index of /arsip yang memuat daftar berkas laporan.pdf dan rahasia1.txt secara interaktif.

<img width="1143" height="842" alt="WhatsApp Image 2026-09-29 at 18 54 47" src="https://github.com/user-attachments/assets/757b41d8-78b2-4965-83c0-420d635f42ad" />

---

## Soal 10
### Perintah soal
1. Jalankan layanan web dinamis (PHP-FPM) pada hostname di node area core (menggunakan Nginx).
2. Buat sebuah aplikasi sederhana yang memuat halaman beranda dan halaman profil.
3. Terapkan aturan rewrite pada server sehingga akses ke /profil dapat berfungsi dengan URL bersih (tanpa akhiran .php).
4. Akses pengujian wajib dilakukan melalui hostname.

### Langkah pengerjaan
**1. Instalasi Nginx & PHP-FPM di Node Core (oblada)**

```Bash
apt-get update
apt-get install -y nginx php8.4-fpm
```

**2. Buat folder aplikasi web core**
```bash
mkdir -p /var/www/html/core
```

**Buat halaman beranda (index.php)**
```bash
cat > /var/www/html/core/index.php <<'EOF'
<!DOCTYPE html>
<html>
<head><title>Beranda Core</title></head>
<body>
    <h1>Selamat Datang di Halaman Beranda Core!</h1>
    <p><a href="/profil">Ke Halaman Profil (Clean URL)</a></p>
</body>
</html>
EOF
```
**Buat halaman profil (profil.php)**
```bash
cat > /var/www/html/core/profil.php <<'EOF'
<!DOCTYPE html>
<html>
<head><title>Profil Kelompok</title></head>
<body>
    <h1>Halaman Profil Kelompok K-59</h1>
    <p>Ini adalah halaman profil dengan URL bersih (tanpa ekstensi .php).</p>
    <p><a href="/">Kembali ke Beranda</a></p>
</body>
</html>
EOF
```
**Konfigurasi Virtual Host Nginx dengan aturan rewrite Clean URL**
```bash
cat > /etc/nginx/sites-available/core <<'EOF'
server {
    listen 80;
    server_name core.k59.com oblada.k59.com molly.k59.com;

    root /var/www/html/core;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ $uri.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
    }
}
EOF

ln -s /etc/nginx/sites-available/core /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
service php8.4-fpm start
service nginx restart
```
**3. Verifikasi dari Klien (alpha / tedd)**
Uji akses halaman beranda dan halaman profil ber-URL bersih via hostname:

```Bash
curl http://oblada.k59.com/
curl http://core.k59.com/
curl http://oblada.k59.com/profil
curl http://core.k59.com/profil
```
Hasil: Beranda menampilkan teks sambutan, dan /profil berhasil memuat halaman profil meskipun diakses tanpa ekstensi .php.

<img width="1113" height="828" alt="image" src="https://github.com/user-attachments/assets/6413ad8e-07ee-4779-b3b0-e1231779e14e" />


---

## Soal 11
### Perintah soal
1. Konfigurasikan Penny (menggunakan Apache) sebagai reverse proxy yang mengarah ke semua node di area vault (obladi & desmond).
2. Konfigurasikan Abbey (menggunakan Nginx) sebagai reverse proxy menuju area core (oblada & molly).
3. Pastikan kedua gerbang ini meneruskan identitas asli pengunjung ke server backend dengan melakukan forwarding header Host dan X-Real-IP

<br><img width="887" height="520" alt="image" src="https://github.com/user-attachments/assets/638309ba-b6b9-4a68-bfb6-33059f0cd74b" /><br>


### Langkah pengerjaan
1. Install apache2 di Penny  
   Soal 11 secara spesifik meminta Penny menggunakan Apache sebagai reverse proxy menuju Obladi dan Desmond
   ```
   apt update
   apt install apache2
   ```
3. Pastikan backend Obladi & Desmond bisa dijangkau Penny  
   Sebelum Apache dikonfigurasi, kita pastikan Penny memang bisa mengakses kedua server backend.
   ```
   ping -c 3 10.93.3.4
   ping -c 3 10.93.3.5
   ```
4. Aktifkan module reverse proxy Apache  
   `proxy` adalah module utama agar Apache bisa bertindak sebagai reverse proxy. `proxy_http` diperlukan karena backend kita diakses menggunakan HTTP. Setelah itu biasanya Apache meminta reload/restart.
   ```
    a2enmod proxy
    a2enmod proxy_http
    a2enmod proxy_balancer
    a2enmod lbmethod_byrequests
    a2enmod headers
   ```
   
    proxy & proxy_http supaya Apache bisa meneruskan HTTP  
    proxy_balancer supaya Apache bisa memiliki beberapa backend  
    lbmethod_byrequests supaya request dibagi berdasarkan jumlah request  
    headers supaya Apache bisa meneruskan/mengatur header
   
   Cek apakah modul sudah aktif
   ```
   apache2ctl -M | grep proxy
   ```
   <br><img width="721" height="296" alt="image" src="https://github.com/user-attachments/assets/cd88a5f0-c21d-44f5-b518-08f4be094b99" /><br>
   Tampilan ini berarti Apache sudah memiliki kemampuan reverse proxy.
6. Buat konfigurasi VirtualHost Penny  
   supaya konfigurasi reverse proxy tidak bercampur dengan konfigurasi Apache bawaan.
   ```
   nano /etc/apache2/sites-available/penny-proxy.conf
   ```
   Isi dengan
   ```
    <VirtualHost *:80>
        ServerName www.k59.com
    
        ProxyPreserveHost On
    
        <Proxy "balancer://vault">
            BalancerMember "http://10.93.3.4"
            BalancerMember "http://10.93.3.5"
            ProxySet lbmethod=byrequests
        </Proxy>
    
        ProxyPass "/" "balancer://vault/"
        ProxyPassReverse "/" "balancer://vault/"
    
        RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
    </VirtualHost>
   ```
   `ProxyPreserveHost On` supaya ketika client mengakses www.k59.com, Apache mempertahankan header Host: www.k59.com
   `RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"` supaya backend mendapatkan IP asli klien

   Kalau nggak buat script gini ajalah, `nano /root/setup-penny-proxy.sh`
   ```
    #!/bin/bash
    
    apt update
    apt install -y apache2
    
    a2enmod proxy
    a2enmod proxy_http
    a2enmod proxy_balancer
    a2enmod lbmethod_byrequests
    a2enmod headers
    
    cat > /etc/apache2/sites-available/penny-proxy.conf <<'EOF'
    <VirtualHost *:80>
        ServerName www.k59.com
    
        ProxyPreserveHost On
    
        <Proxy "balancer://vault">
            BalancerMember "http://10.93.3.4"
            BalancerMember "http://10.93.3.5"
            ProxySet lbmethod=byrequests
        </Proxy>
    
        ProxyPass "/" "balancer://vault/"
        ProxyPassReverse "/" "balancer://vault/"
    
        RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
    </VirtualHost>
    EOF
    
    a2ensite penny-proxy.conf
    
    apache2ctl configtest
    
    if [ $? -eq 0 ]; then
        apache2ctl start
    fi
   ```
   Terus bikin executable
    ```
    chmod +x /root/setup-penny-proxy.sh
    /root/setup-penny-proxy.sh
    ```
   Sebelum,  mengaktifkan config, pastikankita tidak merusak Apache karena typo konfigurasi
   ```
   apache2ctl configtest
   ```
   Kalau hasilnya muncul
   > Syntax OK
   Berarti aman dan bisa langsung aktifkan config
   ```
   a2ensite penny-proxy.conf
   ```
8. Reload apache
   ```
   apache2ctl graceful
   ```
   Pastikan apache berjalan
   ```
   ps aux | grep apache2
   ```
   Nanti bakal muncul proses seperti ini yang menandakan apache aktif
   > /usr/sbin/apache2
   
   Buat request www.k59.com menuju 10.93.4.2 ( penny ) dengan host header tetap www.k59.com
   ```
   curl -I --resolve www.k59.com:80:10.93.4.2 http://www.k59.com
   ```
   <br><img width="693" height="105" alt="image" src="https://github.com/user-attachments/assets/cbf069f8-0ac6-4394-b395-7c1e7991b891" /><br>
   503 Service Unavailable dari reverse proxy berarti Apache Penny menerima request, tetapi tidak berhasil mendapatkan response dari backend Obladi/Desmond. Case apache selesai, kita fokus ke koneksi Penny menuju backend
   ```
   curl -I http://10.93.3.4
   curl -I http://10.93.3.5
   ```
   Case kedua server masih connection failed
   <br><img width="709" height="107" alt="image" src="https://github.com/user-attachments/assets/07096c5b-0689-40d0-b6a2-9c731ec634bf" /><br>
   Apache belum terinstall di obladi & desmond, bikin script aja di obladi & desmond
   ```
   nano /root/setup-vault.sh
   ```
   Isi script obladi
   ```
    #!/bin/bash
    
    apt update
    apt install -y apache2
    
    mkdir -p /var/www/html/arsip
    
    echo "<h1>Vault - Obladi</h1>" > /var/www/html/index.html
    echo "<h1>Arsip - Obladi</h1>" > /var/www/html/arsip/index.html
    
    a2enmod autoindex
    
    apache2ctl configtest
    apache2ctl graceful
   ```
   Buat executable
   ```
   chmod +x /root/setup-vault.sh
   /root/setup-vault.sh
   ```
   Lalu coba
   ```
   curl -I http://localhost
   ```
   Kalau dua duanya (obladi & desmond) berhasil, tampilannya:
   > HTTP/1.1 200 OK
   > Server: Apache/2.4...
9. Kalau kedua backend sudah jalan, balik ke Penny buat ngetes
   ```
    curl -I http://10.93.3.4
    curl -I http://10.93.3.5
    curl -I --resolve www.k59.com:80:10.93.4.2 http://www.k59.com
   ```
   <br><img width="692" height="500" alt="image" src="https://github.com/user-attachments/assets/e7afc529-e6d2-4584-b322-25dda497eef6" /><br>
   Sampai sini sudah membuktikan kalau reverse proxy berhasil, sekarang uji balancer
   ```
    for i in {1..10}; do
        curl -s --resolve www.k59.com:80:10.93.4.2 http://www.k59.com
        echo
    done
   ```
   <br><img width="490" height="285" alt="image" src="https://github.com/user-attachments/assets/3914fc77-f60f-4a95-adb2-143e0d0f9a98" /><br>
   Woooowww kereeennn. Ok lanjut config di obladi buat pembuktian membuktikan forwarding Host dan X-Real-IP. Di obladi & desmos
   ```
   nano /root/proxy-log.conf
   ```
   Isinya
   ```
   LogFormat "%h %l %u %t \"%r\" %>s Host=\"%{Host}i\" X-Real-IP=\"%{X-Real-IP}i\"" proxy_test
   CustomLog /var/log/apache2/proxy_test.log proxy_test
   ```
   Masukkan konfigurasi tersebut ke Apache
   ```
   cat /root/proxy-log.conf >> /etc/apache2/apache2.conf
   ```
   Check
   ```
   apache2ctl configtest
   ```
   Harus muncul syntax OK, and then
   ```
   apache2ctl graceful
   ```
   Lakukan hal yang sama di desmond yeah  
   Now back to Penny
   ```
   curl -s --resolve www.k59.com:80:10.93.4.2 http://www.k59.com
   ```
   Karena Penny menggunakan balancer, request akan masuk ke salah satu dari obladi atau desmond
   <br><img width="463" height="28" alt="image" src="https://github.com/user-attachments/assets/8b114487-7da7-4282-8c11-57d713be5044" /><br>
   Config di obladi untuk membuat VirtualHost yang menangkap request port 80 sekaligus mencatat Host dan X-Real-IP.
    ```
    cat > /root/000-default-vault.conf <<'EOF'
    <VirtualHost *:80>
        ServerName obladi.k59.com
    
        DocumentRoot /var/www/html
    
        LogFormat "%h %l %u %t \"%r\" %>s Host=\"%{Host}i\" X-Real-IP=\"%{X-Real-IP}i\"" proxy_test
        CustomLog /var/log/apache2/proxy_test.log proxy_test
    
        ErrorLog ${APACHE_LOG_DIR}/error.log
    </VirtualHost>
    EOF
    ```
    Jadikan konfigurasi ini sebagai default, yang 000-vault-log.conf di-disable dulu
   ```
   a2dissite 000-vault-log.conf
   a2ensite 000-default.conf
   ```
   Backup config default lama
   ```
   cp /etc/apache2/sites-available/000-default.conf /root/000-default.conf.backup
   ```
   Pasang config baru
   ```
   cp /root/000-default-vault.conf /etc/apache2/sites-available/000-default.conf
   a2ensite 000-default.conf
   apache2ctl configtest
   apache2ctl graceful
   ```
   Tes langsung di obladi
   ```
   curl -H "Host: www.k59.com" \
     -H "X-Real-IP: 10.93.4.2" \
     http://localhost
   ```
   Harus keluar `<h1>Vault - Obladi</h1>`
   Lalu cek
   ```
   tail -n 5 /var/log/apache2/proxy_test.log
   ```
   Targetnya kira kira
   > ::1 - - [30/Sep/2026:...] "GET / HTTP/1.1" 200 ... Host="www.k59.com" X-Real-IP="10.93.4.2"
   Next tes dari Penny
   ```
   curl -H "X-Real-IP: 10.93.4.2" \
     --resolve www.k59.com:80:10.93.4.2 \
     http://www.k59.com
   ```
   <br><img width="292" height="48" alt="image" src="https://github.com/user-attachments/assets/566dfe3c-4cdf-448c-a3e1-410d4f04c3e4" /><br>  
   Lalu cek lagi dari obladi
   ```
   tail -n 5 /var/log/apache2/proxy_test.log
   ```
   <br><img width="497" height="123" alt="image" src="https://github.com/user-attachments/assets/5ced266e-be1d-4223-8884-8812d2a6ab33" /><br>

   Terget keluaran ini membuktikan bahwa Penny meneruskan Host dan X-Real-IP ke backend.
   ```
   Host="www.k59.com"
   X-Real-IP="10.93.4.2"
   ```
   
---

## Soal 12
### Perintah soal
Buat basic Authentication pada node Penny untuk path penny dengan  
username: prabs   
password: pakar_pinter_jadi_gob**

### Langkah pengerjaan
1. Make sure apache jalan
   ```
   service apache2 status
   ```
   Buat direktori /admin
   ```
   mkdir -p /var/www/html/admin
   ```
   Kemudian buat halaman sederhana untuk membuktikan authentication berhasil:
   ```
   echo "<h1>Admin Area - Penny</h1>" > /var/www/html/admin/index.html
   ```
2. Bikin file yg bisa simpan username & password
   ```
   apt-get update
   apt-get install -y apache2-utils
   htpasswd -c /etc/apache2/.htpasswd prabs
   ```
   Masukkan password.
3. Buat konfigurasi authentication
   ```
   nano /var/www/html/admin/.htaccess
   ```
   Isi dengan
   ```
    AuthType Basic
    AuthName "Restricted Admin Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
   ```
    `AuthType Basic` memberitahu Apache bahwa /admin menggunakan Basic Authentication  
    `AuthName "Restricted Admin Area"` memberikan nama/label untuk area yang dilindungi  Biasanya nama ini muncul pada prompt login browser  
    `AuthUserFile /etc/apache2/.htpasswd` memberitahu Apache letak daftar sun & passwd  
    `Require valid-user` artinya hanya user yang memiliki credential valid yang boleh masuk  
4. Pastikan Apache mengizinkan .htaccess karena konfigurasi authentication kita berada di .htaccess.
   ```
   nano /etc/apache2/conf-available/admin-auth.conf
   ```
   Isinya
   ```
   <Directory /var/www/html/admin>
      AllowOverride AuthConfig
      Require all granted
    </Directory>
   ```
   Aktifkan
   ```
   a2enconf admin-auth.conf
   apache2ctl configtest
   apache2ctl graceful
   ```
5. Backup di root
    ```
    nano /root/soal12.sh
    ```
    Isinya
    ```
    #!/bin/bash
    
    apt-get update
    apt-get install -y apache2-utils
    
    mkdir -p /var/www/html/admin
    
    echo "<h1>Admin Area - Penny</h1>" > /var/www/html/admin/index.html
    
    htpasswd -bc /etc/apache2/.htpasswd prabs 'pakar_pinter_jadi_gob**'
    
    cat > /var/www/html/admin/.htaccess <<'EOF'
    AuthType Basic
    AuthName "Restricted Admin Area"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
    EOF
    
    cat > /etc/apache2/conf-available/admin-auth.conf <<'EOF'
    <Directory /var/www/html/admin>
        AllowOverride AuthConfig
        Require all granted
    </Directory>
    EOF
    
    a2enconf admin-auth.conf
    
    apache2ctl configtest
    
    if [ $? -eq 0 ]; then
        apache2ctl graceful
    fi
    ```
    Kemudian
   ```
   chmod +x /root/soal12.sh
   /root/soal12.sh
   ```
6. Testing verifikasi tanpa ke=redensial, misal lewat node alpha
   ```
   curl -i http://penny.k59.com/admin/
   ```
   Harusnya keluar hasil
   > HTTP/1.1 401 Unauthorized
      
   Tapi ini masih keluar
   > curl: (6) Could not resolve host: penny.k59.com (Domain name not found)
     
   artinya Alpha belum bisa menerjemahkan penny.k59.com menjadi IP. Jadi kita cek DNS dulu
   <br>  
   Di alpha, cek `cat /etc/resolv.conf` harus ada nameserver blablabla  
   Perintah `dig +short penny.k59.com` harusnya keluar `10.93.4.2`   
   Kalau dig belum ada  
   ```
   apk add bind-tools
   dig +short penny.k59.com
   ```
   Di prab  
   ```
   ss -lntup | grep :53
   ps aux | grep named
   ```
   Kalau named tidak running  
   ```
   service named start
   service named status
   ss -lntup | grep :53
   dpkg -l | grep bind9
   apt-get update
   DEBIAN_FRONTEND=noninteractive apt-get install -y bind9 bind9-utils dnsutils
   which named
   ls -l /etc/init.d/ | grep -E 'bind|named'
   named-checkconf
   mkdir -p /etc/bind/k59
   ```
   Config ulang, isinya  
   ```
    cat > /etc/bind/k59/k59.com <<'EOF'
    $TTL 604800
    @       IN      SOA     prab.k59.com. root.k59.com. (
                            2026092803
                            604800
                            86400
                            2419200
                            604800 )
    
    @       IN      NS      prab.k59.com.
    @       IN      NS      tedd.k59.com.
    
    @       IN      A       10.93.4.2
    
    prab    IN      A       10.93.3.2
    tedd    IN      A       10.93.3.3
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
    
    vault   IN      A       10.93.3.4
    vault   IN      A       10.93.3.5
    core    IN      A       10.93.3.6
    core    IN      A       10.93.3.7
    
    www     IN      CNAME   penny
    static  IN      CNAME   abbey
    EOF
   ```
   And then  
   ```
   named-checkconf
   named-checkzone k59.com /etc/bind/k59/k59.com
   ```
   Outputnya harus OK, and then  
   ```
   chown -R bind:bind /etc/bind/k59
   service named restart
   service named status
   ```
   Harus `bind is running`  
   Anyway, troubleshoot lagi di prab  
   ```
    cat > /etc/bind/named.conf.local <<'EOF'
    zone "k59.com" {
        type master;
        notify yes;
        also-notify { 10.93.3.3; };
        allow-transfer { 10.93.3.3; };
        file "/etc/bind/k59/k59.com";
    };
    EOF
   ```
   And then  
   ```
   named-checkconf
   named-checkzone k59.com /etc/bind/k59/k59.com
   service named restart
   service named status
   dig @127.0.0.1 penny.k59.com +short
   ```
   Target `10.93.4.2`  
   ```
   dig @127.0.0.1 www.k59.com +short
   ```
   Target  
   > penny.k59.com.
   > 10.93.4.2
     
7. Testing di alpha  
   ```
   dig +short penny.k59.com
   ```
   Target `10.93.4.2`, lalu tes admin tanpa kredensial  
   ```
   curl -i http://penny.k59.com/admin/
   ```
   <br><img width="492" height="301" alt="image" src="https://github.com/user-attachments/assets/96e06439-5ec1-42f1-a769-4ec5cf5d4279" /><br>

   Tes dengan usn & pw benar  
   ```
   curl -i -u 'prabs:pakar_pinter_jadi_gob**' http://penny.k59.com/admin/
   ```
   <br><img width="503" height="136" alt="image" src="https://github.com/user-attachments/assets/095f6349-35b7-4435-b9a4-b561ba3361f6" /><br>
   soal 12 done.    

---

## Soal 13
### Perintah soal
HTTP RedirecT
* **Penny (Apache):** Akses via IP atau `penny.k59.com` di-*redirect* permanen (**HTTP 301**) ke `[www.k59.com](https://www.k59.com)`.
* **Abbey (Nginx):** Akses via IP atau `abbey.k59.com` di-*redirect* sementara (**HTTP 302**) ke `static.k59.com`.
  
### Langkah pengerjaan
1. Script config di node penny, `nano /root/soal13.sh`
    ```
    #!/bin/bash
    
    cat > /etc/apache2/sites-available/penny-proxy.conf <<'EOF'
    <VirtualHost *:80>
        ServerName www.k59.com
        ServerAlias penny.k59.com
    
        RewriteEngine On
    
        RewriteCond %{HTTP_HOST} ^penny\.k59\.com$ [NC]
        RewriteRule ^/(.*)$ http://www.k59.com/$1 [R=301,L]
    
        ProxyPreserveHost On
    
        <Proxy "balancer://vault">
            BalancerMember "http://10.93.3.4"
            BalancerMember "http://10.93.3.5"
            ProxySet lbmethod=byrequests
        </Proxy>
    
        ProxyPass "/" "balancer://vault/"
        ProxyPassReverse "/" "balancer://vault/"
    
        RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
    </VirtualHost>
    
    <VirtualHost *:80>
        ServerName 10.93.4.2
    
        RewriteEngine On
        RewriteRule ^/(.*)$ http://www.k59.com/$1 [R=301,L]
    </VirtualHost>
    EOF
    
    a2enmod rewrite
    
    apache2ctl configtest
    
    if [ $? -eq 0 ]; then
        apache2ctl graceful
    fi
    ```
    and then
   ```
   chmod +x /root/soal13.sh
   /root/soal13.sh
   apache2ctl configtest
   apache2ctl -M | grep rewrite
   ```
2. Test redirect penny
   ```
   curl -I http://penny.k59.com/
   ```
   Target:
   > HTTP/1.1 301 Moved Permanently
   > Location: http://www.k59.com/

   Test IP penny
   ```
   curl -I http://10.93.4.2/
   ```
   Target:
   > HTTP/1.1 301 Moved Permanently
   > Location: http://www.k59.com/

   <br><img width="293" height="161" alt="image" src="https://github.com/user-attachments/assets/b9951fcc-c9f0-4631-892f-4b61b419df29" /><br>
3. Di node abbey
   ```
    apt-get update
    apt-get install -y nginx
    service nginx start
    service nginx status
   ```
   Buat config di root, `nano /root/soal13.sh`
   ```
    #!/bin/bash
    
    cat > /etc/nginx/sites-available/abbey-proxy <<'EOF'
    upstream core_backend {
        server 10.93.3.6;
        server 10.93.3.7;
    }
    
    server {
        listen 80;
        server_name static.k59.com;
    
        location / {
            proxy_pass http://core_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    
    server {
        listen 80;
        server_name abbey.k59.com 10.93.2.2;
    
        return 302 http://static.k59.com$request_uri;
    }
    EOF
    
    rm -f /etc/nginx/sites-enabled/default
    
    ln -sf /etc/nginx/sites-available/abbey-proxy \
           /etc/nginx/sites-enabled/abbey-proxy
    
    nginx -t
    
    if [ $? -eq 0 ]; then
        service nginx reload
    fi
   ```
   And then
   ```
   chmod +x /root/soal13.sh
   /root/soal13.sh
   ```
   Sebelum testing redirect, cek backend Core 10.93.3.6 (Oblada) dan (10.93.3.7) Molly
   ```
   curl -I http://10.93.3.6
   curl -I http://10.93.3.7
   ```
   Troubleshoot backend oblada
   ```
    apt-get update
    apt-get install -y nginx
    service nginx start
    service nginx status
    ss -lntup | grep :80
    ls -l /etc/nginx/sites-available/
   ```
   Hasilnya kira kira `Nginx listen 0.0.0.0:80`
   Ping ping ping
   ```
   ping -c 3 10.93.3.6
   curl -v http://10.93.3.6/
   ```
   Harus `Connected to 10.93.3.6`
   Dari Oblada, cek apakah firewall aktif
   ```
   iptables -L -n -v
   ```
   Cek `ip addr` oblada, harus muncul `10.93.3.6`
   Jalankan command ini dari abbey
   ```
   ping -c 3 10.93.3.6
   curl -v http://10.93.3.6/
   ```
   Ping harus ada reply, curl harus ada `Connected to 10.93.3.6 (10.93.3.6) port 80`
   Tes backend core, masih di abbey
   ```
   curl -I http://10.93.3.6/
   ```
   harus response `http/1.1 200 OK`  
   Now backend molly
   ```
    cat > /root/fix-core.sh <<'EOF'
    #!/bin/bash
    
    set -e
    
    echo "=== STOP NGINX LAMA ==="
    nginx -s stop 2>/dev/null || true
    pkill nginx 2>/dev/null || true
    
    echo "=== START PHP-FPM ==="
    
    if [ -S /run/php/php8.4-fpm.sock ]; then
        echo "PHP-FPM socket sudah ada."
    else
        php-fpm8.4 -D
    fi
    
    echo "=== CEK PHP-FPM SOCKET ==="
    ls -l /run/php/php8.4-fpm.sock
    
    echo "=== TEST NGINX CONFIG ==="
    nginx -t
    
    echo "=== START NGINX ==="
    nginx
    
    echo "=== CEK PORT 80 ==="
    ss -lntup | grep ':80'
    
    echo
    echo "=== TEST LOCAL ==="
    curl -I http://127.0.0.1/
    
    echo
    echo "======================================"
    echo " CORE MOLLY AKTIF"
    echo " IP      : 10.93.3.7"
    echo " NGINX   : PORT 80"
    echo " PHP-FPM : AKTIF"
    echo "======================================"
    EOF
    
    chmod +x /root/fix-core.sh
    /root/fix-core.sh
    ```
   Jangan lupa
   ```
   curl -I http://10.93.3.7/
   ```
   Harus http 200 OK. Tes juga dari abbey
   ```
   curl -I http://10.93.3.7/
   ```
   Kalau http 200 OK berarti backend Core sudah aman dan kita bisa balik menyelesaikan Soal 13
4. Masuk ke soal 13, di abbey :
   ```
    cat > /root/soal13.sh <<'EOF'
    #!/bin/bash
    
    set -e
    
    echo "=== CREATE ABBEY PROXY CONFIG ==="
    
    cat > /etc/nginx/sites-available/abbey-proxy <<'NGINX'
    upstream core_backend {
        server 10.93.3.6;
        server 10.93.3.7;
    }
    
    server {
        listen 80;
        server_name static.k59.com;
    
        location / {
            proxy_pass http://core_backend;
    
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    
    server {
        listen 80;
        server_name abbey.k59.com 10.93.2.2;
    
        return 302 http://static.k59.com$request_uri;
    }
    NGINX
    
    echo "=== ENABLE ABBEY PROXY ==="
    
    rm -f /etc/nginx/sites-enabled/default
    
    ln -sf /etc/nginx/sites-available/abbey-proxy \
           /etc/nginx/sites-enabled/abbey-proxy
    
    echo "=== TEST NGINX ==="
    
    nginx -t
    
    echo "=== RELOAD NGINX ==="
    
    nginx -s reload 2>/dev/null || nginx
    
    echo
    echo "======================================"
    echo " ABBEY PROXY SELESAI"
    echo "======================================"
    EOF
    
    chmod +x /root/soal13.sh
    /root/soal13.sh
   ```
   and then
   ```
   ss -lntup | grep ':80'
   curl -I -H "Host: static.k59.com" http://127.0.0.1/               harus http 200 OK
   curl -I -H "Host: abbey.k59.com" http://127.0.0.1/
   ```
   Harus
   > HTTP/1.1 302 Found  
   > Location: http://static.k59.com/
   
   Cek IP Abbey
   ```
   curl -I http://10.93.2.2/
   ```
   <br><img width="410" height="215" alt="image" src="https://github.com/user-attachments/assets/eb249e92-7349-4336-9d0e-b8bc0e4513d8" /><br>
   Requirement abbey selesai karena sudah redirect 302 dan redirect ke web static
5. Last, cek penny
   ```
   curl -I http://10.93.4.2/
   curl -I -H "Host: penny.k59.com" http://127.0.0.1/
   ```
   <br><img width="396" height="171" alt="image" src="https://github.com/user-attachments/assets/6e74dfd4-3c3d-4a9d-865d-ca85797d5c04" /><br>
   
---

## Soal 14
### Perintah soal
Memastikan access log di semua server web vault dan core mencatat IP asli client, bukan IP gateway Penny atau Abbey  
* Abbey nerima request static, lalu diteruskan ke obladi/desmond
* Penny nerima request vault/core, lalu diteruskan ke oblada/molly
* Intinya: backend harus tahu IP asli client, bukan IP Abbey/Penny
Stepnya cukup atur gateway, terus atur backend, terus tes log

### Langkah pengerjaan
1. Perbaiki obladi & desmond
   Di obladi  
    ```
    a2enmod remoteip
    nano /etc/apache2/conf-available/real-ip.conf
    ```
    Isi dengan
   ```
    RemoteIPHeader X-Real-IP
    RemoteIPInternalProxy 10.93.2.2
   ```
   Lalu perintah
   ```
    a2enconf real-ip
    service apache2 restart
    apache2ctl configtest
   ```
   Di desmon juga! ulangi step yang sama sampai `syntax OK`
   
3. Perbaiki oblada & molly, oblada dulu
   ```
   nano /etc/nginx/nginx.conf
   ```
   Di dalam http { isi gini
   ```
    set_real_ip_from 10.93.2.2;
    real_ip_header X-Real-IP;
   ```
   Lalu
   ```
   nginx -t
   service nginx reload
   ```
   Molly juga digituin ya! TAPI bedanya `set_real_ip_from 10.93.2.2;` jadi `set_real_ip_from 10.93.4.2;`
4. Tes dari alpha
   ```
    curl http://static.k59.com/
    curl http://vault.k59.com/
    curl http://core.k59.com/
   ```
   lalu cek akses log backend
   * Untuk obladi & desmond
     ```
     tail -n 5 /var/log/apache2/access.log
     ```
   <br><img width="566" height="45" alt="image" src="https://github.com/user-attachments/assets/94e39317-6b84-4b64-9377-3aac3709d8f3" /><br>
   <br><img width="572" height="70" alt="image" src="https://github.com/user-attachments/assets/030992e8-ab56-4d96-84e1-084714dade0d" /><br>
   
   * Untuk oblada & molly
     ```
     tail -n 5 /var/log/nginx/access.log
     ```
   <br><img width="570" height="71" alt="image" src="https://github.com/user-attachments/assets/2bfc6a33-ec15-4bbf-8369-ce32299f9570" /><br>
   <br><img width="578" height="82" alt="image" src="https://github.com/user-attachments/assets/35c9b57b-0ea1-456c-ac74-050c22cfbd10" /><br>

---

## Soal 15
### Perintah soal
Soal ini meminta bikin dua path website tambahan:
* Di Penny kita harus buat /eternal yang diarahkan oleh Apache ke backend /var/www/eternal, dan PHP harus bisa dirender.
* Di Abbey kita harus buat /orion yang diarahkan ke /var/www/orion, tetapi hanya file statis, jadi tidak menggunakan PHP.
  
### Langkah pengerjaan
1. Di penny, cek apache
   ```
   apache2ctl -v
   ```
   Buat folder backend
   ```
   mkdir -p /var/www/eternal
   ```
   Buat halaman php untuk testing
   ```
    cat > /var/www/eternal/index.php <<'EOF'
    <?php
    echo "<h1>Eternal</h1>";
    echo "<p>PHP rendering berhasil.</p>";
    echo "<p>Server: " . $_SERVER['SERVER_NAME'] . "</p>";
    ?>
    EOF
   ```
   Install php FM
   ```
    apt-get update
    apt-get install -y php php-fpm
    apt-get install -y php8.4-fpm
    service php8.4-fpm start
   ```
   Aktifkan module yang dibutuhkan PHP-FPM
   ```
   a2enmod proxy proxy_fcgi setenvif
   ```
   Config PHP-FPM eksternal
   ```
    cat > /etc/apache2/conf-available/eternal-php.conf <<'EOF'
    <Directory /var/www/eternal>
        Require all granted
        DirectoryIndex index.php index.html
    
        <FilesMatch "\.php$">
            SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost/"
        </FilesMatch>
    </Directory>
    EOF
   ```
   Aktifkan config
   ```
   a2enconf eternal-php
   apache2ctl configtest
   ```
2. Hubungkan /eternal ke folder /var/www/eternal di Apache Penny
   Config Virtualhost
   ```
    cat > /etc/apache2/sites-available/eternal.conf <<'EOF'
    <VirtualHost *:80>
        ServerName penny.k59.com
    
        Alias /eternal/ /var/www/eternal/
    
        <Directory /var/www/eternal>
            Require all granted
            DirectoryIndex index.php index.html
        </Directory>
    </VirtualHost>
    EOF
   ```
   Maksudnya: ketika orang mengakses `http://penny.k59.com/eternal/`, Apache mengambil file dari `/var/www/eternal/`
   Aktifkan site
   ```
   a2ensite eternal.conf
   apache2ctl configtest
   ```
   Restart apache
   ```
   service apache2 restart
   ```
   Di penny, jalankan
   ```
   curl -H "Host: penny.k59.com" http://127.0.0.1/eternal/
   ```
   Kalau berhasil, harus keluar hasil HTML seperti:
    ```
    <h1>Eternal</h1>
    <p>PHP rendering berhasil.</p>
    ```
    <br><img width="501" height="28" alt="image" src="https://github.com/user-attachments/assets/796df889-1228-4faa-97b9-cd60ee1f6fee" /><br>
    bagian Penny /eternal sudah selesai  
3. Pembuatan /var/www/orion dan konfigurasi static /orion
   Di abbey, buat folder orion
   ```
   mkdir -p /var/www/orion
   ```
   Buat file html statis
   ```
    cat > /var/www/orion/index.html <<'EOF'
    <!DOCTYPE html>
    <html>
    <head>
        <title>Orion</title>
    </head>
    <body>
        <h1>Orion</h1>
        <p>Static content berhasil.</p>
    </body>
    </html>
    EOF
   ```
   Tambahkan config abey-proxy
   ```
    cat > /etc/nginx/sites-available/abbey-proxy <<'EOF'
    upstream core_backend {
        server 10.93.3.6;
        server 10.93.3.7;
    }
    
    server {
        listen 80;
        server_name static.k59.com;
    
        location / {
            proxy_pass http://core_backend;
    
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    
    server {
        listen 80;
        server_name abbey.k59.com 10.93.2.2;
    
        location /orion/ {
            alias /var/www/orion/;
            index index.html;
        }
    
        location = /orion {
            return 301 /orion/;
        }
    
        location / {
            return 302 http://static.k59.com$request_uri;
        }
    }
    EOF
   ```
   Cek config nginx
   ```
   nginx -t
   ```
   Restart nginx
   ```
   service nginx restart
   ```
   Test /orion
   ```
   curl -H "Host: abbey.k59.com" http://127.0.0.1/orion/
   ```
   <br><img width="432" height="196" alt="image" src="https://github.com/user-attachments/assets/22ecccc5-2ea7-4359-bb96-f1ce21d7c3a5" /><br>
   File html berhasil muncul. Soal 15 selesai

   

---

## Soal 16
### Perintah soal
Jadi, **Soal 16 itu intinya nyuruh kita menguji seberapa kuat/performa web server kita ketika menerima banyak request secara bersamaan.**

Bayangin kita punya dua pintu masuk website:  
* `www.k59.com` → masuk ke **Penny**
* `static.k59.com` → masuk ke **Abbey**

Nah, kita pura-pura jadi **250 orang yang melakukan request ke website**, tetapi request-nya dibuat secara bersamaan dengan **10 koneksi aktif dalam satu waktu**  

Caranya menggunakan **ApacheBench (`ab`)** dari salah satu client, misalnya Alpha  

Setelah itu kita **lihat hasil pengujiannya**, misalnya:
* berapa request yang berhasil,
* berapa lama total pengujian,
* berapa request per detik yang bisa dilayani,
* berapa lama rata-rata server merespons,
* apakah ada request yang gagal.

Jadi sederhananya:
> **Soal 16 meminta kita melakukan simulasi beban terhadap dua website yang sudah kita bangun, lalu mencatat performa masing-masing server ketika menerima 250 request dengan 10 request berjalan bersamaan.**

### Langkah pengerjaan
1. Di alpha, cek apache bench
   ```
    apk update
    apk add apache2-utils
   ```
   Benchmark www.k59.com
   ```
   ab -n 250 -c 10 http://www.k59.com/
   ```
   * -n 250 → total 250 request
   * -c 10 → maksimal 10 request berjalan bersamaan
   * http://www.k59.com/ → website yang diuji
   
   <br><img width="456" height="594" alt="image" src="https://github.com/user-attachments/assets/8230ffbc-8025-41ca-862d-5305274111f6" /><br>

   250 request dikirim
   0 gagal koneksi
   0 gagal menerima response
   125 response punya ukuran konten yang berbeda dari ukuran yang dianggap ApacheBench sebagai normal  

   Performa yang tercatat:
   * Requests per second: 2149.41
   * Time per request:    4.652 ms
  
   Pengujian kedua
   ```
   ab -n 250 -c 10 http://static.k59.com/
   ```
   <br><img width="526" height="567" alt="image" src="https://github.com/user-attachments/assets/d07b3867-cfa6-4e24-864e-db3f439f207c" /><br>

   **Perbandingan Hasil Benchmark**

| Parameter | [www.k59.com](https://www.google.com/search?q=https%3A%2F%2Fwww.k59.com) | static.k59.com |
| --- | --- | --- |
| **Server** | Apache (Penny) | Nginx (Abbey) |
| **Requests** | 250 | 250 |
| **Concurrency** | 10 | 10 |
| **Failed Requests** | 125 (Length) | 124 (Length) |
| **Requests / sec** | 2149.41 | 1946.57 |
| **Time / request** | 4.652 ms | 5.137 ms |
| **Response Size** | 24 bytes | 615 bytes |

**Analisis Sederhana**
* Pengujian dilakukan dari klien **Alpha** menggunakan ApacheBench (`ab`) dengan **250 request** dan tingkat konkurensi **10** ke kedua website.
* **[www.k59.com](https://www.google.com/search?q=https%3A%2F%2Fwww.k59.com)** (dilayani Apache di Penny) mencatat sekitar **2.149,41 request/detik** dengan rata-rata waktu **4,652 ms** per request.
* **static.k59.com** (dilayani Nginx di Abbey) mencatat sekitar **1.946,57 request/detik** dengan rata-rata waktu **5,137 ms** per request.
* **Catatan Failed Requests:** Angka *failed* yang muncul pada kedua server murni disebabkan oleh perbedaan panjang respons (*Length*), bukan karena kegagalan koneksi (*Connect* atau *Receive*). Ini berarti seluruh koneksi berhasil terhubung dengan sempurna.

**Kesimpulan**
Pengujian performa menggunakan ApacheBench menunjukkan bahwa kedua server berhasil menerima dan memproses seluruh koneksi dengan stabil. [www.k59.com](https://www.google.com/search?q=https%3A%2F%2Fwww.k59.com) mencatatkan throughput 2149,41 request/detik dengan latensi rata-rata 4,652 ms, sedangkan static.k59.com mencatatkan 1946,57 request/detik dengan latensi rata-rata 5,137 ms. Terdapat sejumlah *failed requests* pada keduanya yang seluruhnya dikategorikan sebagai perbedaan panjang respons (*Length*), bukan kegagalan koneksi.

---

## Soal 17
### Perintah soal
* Tambahkan TXT record pada DNS untuk semua klien sayap kiri dan sayap kanan (Alpha, Beta, Gamma, Delta, Epsilon).
* Jika DNS di-query TXT terhadap nama domain mereka (contoh: alpha.<xxxx>.com), sistem harus mengembalikan teks berupa nama hostname mereka masing-masing (contoh: "alpha").
### Langkah pengerjaan

1. Master DNS server adalah prab, sehingga, sehingga kita perlu masuk ke prab  
   Cek zone file
   ```
   grep -E '^(alpha|beta|gamma|delta|epsilon)' /etc/bind/k59/k59.com
   ```
   <br><img width="466" height="71" alt="image" src="https://github.com/user-attachments/assets/875850bb-de7d-4b30-8ed3-2bc5f83beac0" /><br>
   Tambahkan TXT record ke zone file yang sudah ada
   ```
   nano /etc/bind/k59/k59.com
   ```
   Tambahkan
   ```
    alpha   IN      TXT     "alpha"
    beta    IN      TXT     "beta"
    gamma   IN      TXT     "gamma"
    delta   IN      TXT     "delta"
    epsilon IN      TXT     "epsilon"
   ```
   Cek serial SOA
   ```
   grep -A1 'SOA' /etc/bind/k59/k59.com
   ```
   <br><img width="330" height="35" alt="image" src="https://github.com/user-attachments/assets/55031d36-b548-4f5e-a333-78085d438875" /><br>
   Buka
   ```
   nano /etc/bind/k59/k59.com
   ```
   Cek zoned file, output : OK
   ```
   named-checkzone k59.com /etc/bind/k59/k59.com
   ```
   Reload DNS di prab, cek tiap tiap node
   ```
   rndc reload
   
    for host in alpha beta gamma delta epsilon; do
        echo -n "$host: "
        dig @10.93.3.2 "$host.k59.com" TXT +short
    done
   ```
   
   <br><img width="546" height="158" alt="image" src="https://github.com/user-attachments/assets/20901953-00b0-4d12-9e03-7a5dd55422f4" /><br>
2. Cek TXT record di Slave (Tedd), masih di prab, query langsung ke DNS slave
   ```
    for host in alpha beta gamma delta epsilon; do
        echo -n "$host: "
        dig @10.93.3.3 "$host.k59.com" TXT +short
    done
   ```
   Jika DNS service tedd tidak menerima koneksi
   ```
   apt-get update
   DEBIAN_FRONTEND=noninteractive apt-get install -y bind9 bind9-utils dnsutils
   which named
   ls -l /etc/init.d/ | grep -E 'bind|named'
   named-checkconf
   service named start
   service named status
   ```
   Jika bind sudah jalan, Tambahkan zone slave di Tedd
   ```
   nano /etc/bind/named.conf.local
   ```
   Tambahkan ini di bagian paling bawah
   ```
    zone "k59.com" {
        type slave;
        masters { 10.93.3.2; };
        file "/var/lib/bind/k59.com";
    };
   ```
   Cek konfig
   ```
   named-checkconf
   service named reload
   ls -l /var/lib/bind/k59.com
   ```
   Verifikasi TXT di Tedd. Sekarang di tedd jalankan:
   ```
    for host in alpha beta gamma delta epsilon; do
        echo -n "$host: "
        dig @10.93.3.3 "$host.k59.com" TXT +short
    done
   ```
   <br><img width="467" height="144" alt="image" src="https://github.com/user-attachments/assets/1aeb7c9e-749b-47d4-8784-08663b949355" /><br>
   Kelima nama host sudah muncul ketika dipanggil, sengan ini nomor 17 selesai.
   
---

## Soal 18

    


   
---

## Soal 19
### Perintah soal
### Langkah pengerjaan

---

## Soal 20
### Perintah soal
### Langkah pengerjaan

---

## Soal 12 (Revisi)
### Perintah soal
Buat basic Authentication pada node Penny untuk path `/admin` dengan  
username: `prabs`  
password: `pakar_pinter_jadi_gob**`

### Konsep singkat
**Basic Authentication** itu seperti satpam di depan sebuah ruangan. Setiap pengunjung yang masuk ke path `/admin` ditanya username dan password. Kalau cocok dengan daftar, boleh masuk (`200 OK`). Kalau tidak, ditolak (`401 Unauthorized`).

Ada tiga bagian yang saling terhubung:
| Bagian | Fungsinya | Lokasi |
| --- | --- | --- |
| Daftar username & password | Buku tamu yang dipegang satpam. Password disimpan dalam bentuk hash, bukan teks asli | `/etc/apache2/.htpasswd` |
| `.htaccess` | Aturan "folder ini dijaga, cek buku tamu yang mana" | `/var/www/html/admin/.htaccess` |
| `AllowOverride AuthConfig` | Izin supaya Apache mau membaca aturan di `.htaccess`. Tanpa izin ini, `.htaccess` diabaikan dan folder tidak terkunci | `/etc/apache2/conf-available/admin-auth.conf` |

> **PENTING:** semua langkah soal 12 dikerjakan di **penny**, bukan di prab. Cek prompt harus `root@penny:~#` sebelum mengetik apa pun.

### Langkah pengerjaan
1. Pastikan Apache jalan di penny
   ```
   service apache2 status
   ```
   Harus muncul tanda Apache sedang berjalan. Kalau belum jalan: `apache2ctl start`

   > **Error 1 (salah node):** saat pertama dikerjakan, perintah dijalankan di **prab**, bukan penny. Hasilnya:
   > ```
   > root@prab:~# service apache2 status
   > grep: /etc/init.d/apache2: No such file or directory
   > apache2: unrecognized service
   > ```
   > **Penyebab:** prab adalah server DNS (BIND), paket `apache2` tidak pernah terinstal di sana, jadi skrip service-nya tidak ada.  
   > **Perbaikan:** pindah ke console penny, lalu ulangi dari langkah 1.

2. Buat folder `/admin` dan halaman pembuktian
   ```
   mkdir -p /var/www/html/admin
   echo "<h1>Admin Area - Penny</h1>" > /var/www/html/admin/index.html
   ```
   `mkdir -p` membuat folder, dan tidak error kalau foldernya sudah ada.

   > **Error 2 (salah node):**
   > ```
   > root@prab:~# echo "<h1>Admin Area - Penny</h1>" > /var/www/html/admin/index.html
   > bash: /var/www/html/admin/index.html: No such file or directory
   > ```
   > **Penyebab:** folder `/var/www/html/` dibuat oleh paket `apache2`. Di prab paketnya tidak ada, jadi folder induknya pun tidak ada. Selain itu langkah `mkdir` belum dijalankan.  
   > **Perbaikan:** jalankan di penny, dan `mkdir -p` dulu sebelum `echo`.

3. Buat file daftar username & password
   ```
   apt-get update
   apt-get install -y apache2-utils
   htpasswd -c /etc/apache2/.htpasswd prabs
   ```
   Saat diminta, ketik password `pakar_pinter_jadi_gob**` (dua kali, tidak tampil di layar).  
   `htpasswd` adalah alat pembuat file daftar user. Opsi `-c` artinya *create* (buat file baru). Pakai `-c` **hanya sekali**. Kalau dipakai lagi untuk menambah user, seluruh isi file lama ditimpa.

   > **Error 3 (salah node):**
   > ```
   > root@prab:~# htpasswd -c /etc/apache2/.htpasswd prabs
   > htpasswd: cannot create file /etc/apache2/.htpasswd
   > ```
   > **Penyebab:** `apache2-utils` sudah terinstal (makanya perintah `htpasswd` bisa dijalankan), tetapi paket itu hanya berisi alat bantu. Folder `/etc/apache2/` dibuat oleh paket `apache2`, yang tidak ada di prab. `htpasswd` tidak bisa membuat file di folder yang tidak ada.  
   > **Perbaikan:** jalankan di penny, karena `apache2` sudah terinstal di sana sejak soal 11.

4. Buat konfigurasi authentication di `.htaccess`
   ```
   nano /var/www/html/admin/.htaccess
   ```
   Isi dengan:
   ```
   AuthType Basic
   AuthName "Restricted Admin Area"
   AuthUserFile /etc/apache2/.htpasswd
   Require valid-user
   ```
   | Baris | Fungsinya |
   | --- | --- |
   | `AuthType Basic` | Memberitahu Apache bahwa folder ini memakai Basic Authentication |
   | `AuthName "Restricted Admin Area"` | Label area yang dilindungi. Teks ini muncul di kotak login browser |
   | `AuthUserFile /etc/apache2/.htpasswd` | Letak file daftar username & password |
   | `Require valid-user` | Hanya user yang kredensialnya ada di daftar yang boleh masuk |

5. Izinkan Apache membaca `.htaccess`
   ```
   nano /etc/apache2/conf-available/admin-auth.conf
   ```
   Isi dengan:
   ```
   <Directory /var/www/html/admin>
       AllowOverride AuthConfig
       Require all granted
   </Directory>
   ```
   `AllowOverride AuthConfig` mengizinkan `.htaccess` mengatur autentikasi, dan hanya autentikasi. Aktifkan:
   ```
   a2enconf admin-auth.conf
   apache2ctl configtest
   apache2ctl graceful
   ```
   `configtest` harus menampilkan `Syntax OK` sebelum `graceful` (reload tanpa memutus koneksi).

6. Simpan semua langkah sebagai script backup
   ```
   nano /root/soal12.sh
   ```
   Isinya:
   ```
   #!/bin/bash

   apt-get update
   apt-get install -y apache2-utils

   mkdir -p /var/www/html/admin

   echo "<h1>Admin Area - Penny</h1>" > /var/www/html/admin/index.html

   htpasswd -bc /etc/apache2/.htpasswd prabs 'pakar_pinter_jadi_gob**'

   cat > /var/www/html/admin/.htaccess <<'EOF'
   AuthType Basic
   AuthName "Restricted Admin Area"
   AuthUserFile /etc/apache2/.htpasswd
   Require valid-user
   EOF

   cat > /etc/apache2/conf-available/admin-auth.conf <<'EOF'
   <Directory /var/www/html/admin>
       AllowOverride AuthConfig
       Require all granted
   </Directory>
   EOF

   a2enconf admin-auth.conf

   apache2ctl configtest

   if [ $? -eq 0 ]; then
       apache2ctl graceful
   fi
   ```
   Lalu jalankan:
   ```
   chmod +x /root/soal12.sh
   /root/soal12.sh
   ```
   Di script, `htpasswd -bc` memasukkan password langsung di perintah (`-b` = *batch*), jadi tidak ada prompt. Password ditulis dalam tanda kutip tunggal `'...'` supaya karakter `**` dibaca sebagai teks biasa oleh bash, bukan sebagai wildcard.  
   Pastikan baris `EOF` di script **rata kiri** (tanpa spasi di depan), kalau tidak heredoc tidak berhenti dan isi perintah berikutnya ikut tertulis ke file.

7. Pastikan klien bisa menerjemahkan `penny.k59.com` (cek DNS dulu)
   Di alpha:
   ```
   cat /etc/resolv.conf
   dig +short penny.k59.com
   ```
   Harus keluar `10.93.4.2`. Kalau `dig` belum ada: `apk add bind-tools`

   > **Error 4 (DNS belum jalan):** saat tes pertama dari alpha:
   > ```
   > curl: (6) Could not resolve host: penny.k59.com (Domain name not found)
   > ```
   > **Penyebab:** alpha tidak bisa menerjemahkan nama `penny.k59.com` menjadi IP. Ini bukan masalah autentikasi. Penyebabnya ada di prab: BIND (`named`) mati atau belum terinstal, sehingga tidak ada yang menjawab pertanyaan DNS.  
   > **Perbaikan (di prab):**
   > ```
   > service named start
   > service named status
   > ss -lntup | grep :53
   > ```
   > Kalau `named` tidak ada sama sekali:
   > ```
   > apt-get update
   > DEBIAN_FRONTEND=noninteractive apt-get install -y bind9 bind9-utils dnsutils
   > which named
   > named-checkconf
   > ```
   > Lalu tulis ulang zona `/etc/bind/k59/k59.com` (dengan serial yang naik) dan `/etc/bind/named.conf.local`, cek, dan restart:
   > ```
   > named-checkzone k59.com /etc/bind/k59/k59.com
   > chown -R bind:bind /etc/bind/k59
   > service named restart
   > dig @127.0.0.1 penny.k59.com +short
   > ```
   > Target: `10.93.4.2`. Setelah itu `dig +short penny.k59.com` dari alpha harus keluar `10.93.4.2`.

8. Pengujian dari alpha

   **Tanpa kredensial**, harus ditolak:
   ```
   curl -i http://penny.k59.com/admin/
   ```
   Target:
   > HTTP/1.1 401 Unauthorized  
   > WWW-Authenticate: Basic realm="Restricted Admin Area"

   **Dengan kredensial benar**, harus diterima:
   ```
   curl -i -u 'prabs:pakar_pinter_jadi_gob**' http://penny.k59.com/admin/
   ```
   Target:
   > HTTP/1.1 200 OK  
   > `<h1>Admin Area - Penny</h1>`

   `-u 'user:password'` membuat curl mengirim kredensial Basic Authentication. Tanda kutip tunggal menjaga `**` tetap teks biasa.

   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/3818743d-007b-41bd-9158-580852042cf4" /><br>
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/4962d124-6315-4e86-8230-1cc05e6bb278" /><br>

   Soal 12 selesai.

### Rangkuman error & perbaikan
| No | Gejala | Penyebab | Perbaikan |
| --- | --- | --- | --- |
| 1 | `service apache2 status`: `No such file or directory`, `unrecognized service` | Dikerjakan di prab (server DNS), Apache tidak terinstal di sana | Pindah ke penny |
| 2 | `echo ... > /var/www/html/admin/index.html`: `No such file or directory` | Folder `/var/www/html` tidak ada di prab, dan `mkdir` belum dijalankan | Kerjakan di penny, `mkdir -p` dulu |
| 3 | `htpasswd: cannot create file /etc/apache2/.htpasswd` | Folder `/etc/apache2` tidak ada di prab. `apache2-utils` hanya berisi alat, folder dibuat oleh paket `apache2` | Kerjakan di penny |
| 4 | `curl: (6) Could not resolve host: penny.k59.com` | BIND di prab mati atau belum terinstal, DNS tidak menjawab | Nyalakan atau instal BIND di prab, perbaiki zona, tes dengan `dig` |

### Pelajaran
* Error 1 sampai 3 punya akar masalah yang sama: **salah node**. Tiap node di GNS3 adalah komputer terpisah, jadi paket dan folder di satu node tidak ada di node lain. Biasakan melihat prompt (`root@namanode`) sebelum menjalankan perintah.
* Error `No such file or directory` biasanya berarti folder induknya tidak ada, dan folder induk itu sering dibuat oleh paket yang belum terinstal.
* Kalau `curl` gagal dengan `Could not resolve host`, masalahnya di DNS, bukan di web server. Tes dengan `dig` dulu sebelum menyalahkan Apache.

### Catatan
Pengujian soal 12 dilakukan sebelum soal 13. Setelah soal 13, `penny.k59.com` di-redirect permanen (301) ke `www.k59.com`, jadi kalau tes `curl http://penny.k59.com/admin/` diulang setelahnya, hasilnya bisa berupa redirect, bukan `401`. Bukti `401` dan `200` di atas tetap valid karena diambil sebelum aturan redirect dipasang.

---

## Soal 15 (Revisi)
### Perintah soal
Buat dua path website tambahan:
* Di **Penny (Apache)**: path `/eternal` diarahkan ke folder backend `/var/www/eternal`, dan **PHP harus bisa dirender**.
* Di **Abbey (Nginx)**: path `/orion` diarahkan ke folder `/var/www/orion`, tetapi **hanya file statis**, tidak memakai PHP.

### Konsep singkat
Apache atau Nginx bisa melayani sebuah path dengan dua cara:
1. **Ambil file dari folder sendiri.** Di Apache dilakukan oleh `Alias`, di Nginx oleh `root`.
2. **Teruskan ke server lain (proxy).** Di Apache dilakukan oleh `ProxyPass`, di Nginx oleh `proxy_pass`.

Penny dan Abbey sudah menjadi reverse proxy (soal 11 dan 13), jadi secara bawaan semua path mereka diteruskan ke backend. Supaya `/eternal` dan `/orion` dilayani oleh Penny dan Abbey sendiri, dua path itu harus **dikecualikan** dari proxy.

Untuk PHP: Apache hanya bisa mengirim file. Yang menjalankan kode PHP adalah **PHP-FPM**. Apache menyerahkan file `.php` ke PHP-FPM lewat modul `proxy_fcgi` dan sebuah socket, lalu mengirim hasil HTML-nya ke pengunjung. Di Abbey PHP-FPM sengaja tidak dipasang karena soal meminta static only.

> **PENTING:** Bagian 1 dikerjakan di **penny**, Bagian 2 di **abbey**. Cek prompt (`root@penny` atau `root@abbey`) sebelum mengetik perintah.

---

## Bagian 1: Penny (`/eternal`, Apache + PHP)

### Langkah pengerjaan
1. Install dan nyalakan PHP-FPM
   ```
   apt-get update
   apt-get install -y php8.4-fpm
   service php8.4-fpm start
   ls -l /run/php/php8.4-fpm.sock
   ```
   * `php8.4-fpm` dipakai agar versinya sama dengan soal 10.
   * File `.sock` adalah "pintu telepon" internal antara Apache dan PHP-FPM. Kalau filenya muncul, PHP-FPM sudah hidup.
   * Tidak memakai `apt-get install php` biasa, karena paket itu menarik `libapache2-mod-php` yang bisa bentrok dengan `proxy_fcgi`.

2. Buat folder backend dan halaman PHP untuk tes
   ```
   mkdir -p /var/www/eternal
   cat > /var/www/eternal/index.php <<'EOF'
   <?php
   echo "<h1>Eternal</h1>";
   echo "<p>PHP rendering berhasil.</p>";
   ?>
   EOF
   ```

3. Aktifkan modul Apache
   ```
   a2enmod proxy proxy_fcgi setenvif rewrite
   ```
   `proxy_fcgi` menghubungkan Apache ke PHP-FPM, dan `rewrite` dipakai oleh redirect soal 13.

4. Buat aturan PHP khusus folder eternal
   ```
   cat > /etc/apache2/conf-available/eternal-php.conf <<'EOF'
   <Directory /var/www/eternal>
       Require all granted
       DirectoryIndex index.php index.html
       <FilesMatch "\.php$">
           SetHandler "proxy:unix:/run/php/php8.4-fpm.sock|fcgi://localhost/"
       </FilesMatch>
   </Directory>
   EOF
   a2enconf eternal-php
   ```
   | Baris | Fungsinya |
   | --- | --- |
   | `<Directory /var/www/eternal>` | Aturan hanya berlaku untuk folder ini |
   | `Require all granted` | Semua orang boleh membuka folder. Tanpa ini hasilnya 403 |
   | `DirectoryIndex index.php index.html` | Kalau yang dibuka folder, file pertama yang dicari adalah `index.php`, lalu `index.html` |
   | `<FilesMatch "\.php$">` | Berlaku hanya untuk file berakhiran `.php` |
   | `SetHandler "proxy:unix:...\|fcgi://localhost/"` | File `.php` diserahkan ke PHP-FPM lewat socket |

5. Gabungkan `/eternal` ke `penny-proxy.conf`
   Nonaktifkan vhost terpisah (kalau sebelumnya sempat dibuat), lalu tulis ulang `penny-proxy.conf`. Isinya sudah menggabungkan soal 11, 13, dan 15:
   ```
   a2dissite eternal.conf

   cat > /etc/apache2/sites-available/penny-proxy.conf <<'EOF'
   <VirtualHost *:80>
       ServerName www.k59.com
       ServerAlias penny.k59.com

       RewriteEngine On
       RewriteCond %{HTTP_HOST} ^penny\.k59\.com$ [NC]
       RewriteCond %{REQUEST_URI} !^/eternal
       RewriteRule ^/(.*)$ http://www.k59.com/$1 [R=301,L]

       Alias /eternal /var/www/eternal

       ProxyPreserveHost On
       ProxyPass /eternal !

       <Proxy "balancer://vault">
           BalancerMember "http://10.93.3.4"
           BalancerMember "http://10.93.3.5"
           ProxySet lbmethod=byrequests
       </Proxy>

       ProxyPass "/" "balancer://vault/"
       ProxyPassReverse "/" "balancer://vault/"

       RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
   </VirtualHost>

   <VirtualHost *:80>
       ServerName 10.93.4.2
       RewriteEngine On
       RewriteRule ^/(.*)$ http://www.k59.com/$1 [R=301,L]
   </VirtualHost>
   EOF
   ```
   Yang baru dibanding soal 13:
   * `Alias /eternal /var/www/eternal`: URL `/eternal` mengambil file dari folder itu. Ditulis tanpa garis miring di belakang supaya `/eternal` dan `/eternal/` sama-sama cocok.
   * `ProxyPass /eternal !`: tanda `!` artinya "jangan diproxy". Baris ini **harus ditulis sebelum** `ProxyPass "/"`, karena Apache memakai aturan pertama yang cocok.
   * `RewriteCond %{REQUEST_URI} !^/eternal`: redirect 301 soal 13 tidak berlaku untuk `/eternal`. Path lain tetap di-redirect.

6. Terapkan konfigurasi
   ```
   apache2ctl configtest
   apache2ctl graceful
   ```
   `configtest` harus menampilkan `Syntax OK` sebelum `graceful`.

7. Pengujian di penny
   ```
   curl -i -H "Host: www.k59.com" http://127.0.0.1/eternal/
   curl -I -H "Host: penny.k59.com" http://127.0.0.1/eternal/
   curl -I -H "Host: penny.k59.com" http://127.0.0.1/
   ```
   Target:
   * Dua tes pertama: `HTTP/1.1 200 OK` dengan isi `<h1>Eternal</h1>` dan `<p>PHP rendering berhasil.</p>`.
   * Tes ketiga: `HTTP/1.1 301 Moved Permanently` ke `www.k59.com`, menandakan redirect soal 13 tidak rusak.

   Kalau yang muncul di browser atau curl adalah teks `<?php echo ... ?>`, berarti PHP belum dirender. Cek `ls -l /run/php/php8.4-fpm.sock` dan `a2query -c eternal-php`.

   Dari alpha:
   ```
   curl http://www.k59.com/eternal/
   ```

   <br><img width="501" height="28" alt="image" src="https://github.com/user-attachments/assets/796df889-1228-4faa-97b9-cd60ee1f6fee" /><br>

---

## Bagian 2: Abbey (`/orion`, Nginx static)

### Langkah pengerjaan
1. Buat folder dan halaman statis
   ```
   mkdir -p /var/www/orion
   cat > /var/www/orion/index.html <<'EOF'
   <!DOCTYPE html>
   <html>
   <head><title>Orion</title></head>
   <body>
       <h1>Orion</h1>
       <p>Static content berhasil.</p>
   </body>
   </html>
   EOF
   ```

2. Tulis ulang config Nginx dengan `location /orion/`
   Blok `/orion` ditaruh di **kedua** server block (`static.k59.com` dan `abbey.k59.com`), supaya bisa diakses dari kedua nama:
   ```
   cat > /etc/nginx/sites-available/abbey-proxy <<'EOF'
   upstream core_backend {
       server 10.93.3.6;
       server 10.93.3.7;
   }

   server {
       listen 80;
       server_name static.k59.com;

       location = /orion { return 301 /orion/; }
       location /orion/ {
           root /var/www;
           index index.html;
       }
       location ~* ^/orion/.*\.php$ { return 403; }

       location / {
           proxy_pass http://core_backend;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }

   server {
       listen 80;
       server_name abbey.k59.com 10.93.2.2;

       location = /orion { return 301 /orion/; }
       location /orion/ {
           root /var/www;
           index index.html;
       }
       location ~* ^/orion/.*\.php$ { return 403; }

       location / {
           return 302 http://static.k59.com$request_uri;
       }
   }
   EOF
   ```
   | Baris | Fungsinya |
   | --- | --- |
   | `location = /orion { return 301 /orion/; }` | `/orion` tanpa garis miring diarahkan ke `/orion/` |
   | `location /orion/ { ... }` | Semua URL berawalan `/orion/` masuk ke blok ini |
   | `root /var/www;` | Nginx menyambung `root` + URL, jadi `/orion/index.html` dibaca dari `/var/www/orion/index.html` |
   | `index index.html;` | File default kalau yang dibuka folder |
   | `location ~* ^/orion/.*\.php$ { return 403; }` | File `.php` di orion ditolak, jadi benar-benar static only |

   Nginx memilih `location` dengan awalan terpanjang, jadi `/orion/` menang atas `/`. Redirect 302 soal 13 tetap berjalan untuk path lain.

3. Cek config, lalu pastikan Nginx hidup
   ```
   nginx -t
   service nginx start
   ss -lntup | grep ':80'
   ```
   Harus ada baris `0.0.0.0:80` dengan nama `nginx`. Kalau Nginx sudah hidup dan hanya perlu membaca config baru, pakai `service nginx reload`.

   > **Error 1 (Nginx belum menyala):** saat tes pertama di abbey:
   > ```
   > root@abbey:~# curl -i -H "Host: abbey.k59.com" http://127.0.0.1/orion/
   > curl: (7) Failed to connect to 127.0.0.1 port 80 after 0 ms: Could not connect to server
   > ```
   > **Penyebab:** Nginx belum berjalan, jadi tidak ada proses yang mendengarkan di port 80. Pesan `connection refused` (gagal dalam 0 ms) menandakan koneksi ditolak langsung. Kalau masalahnya path atau file, jawabannya `404`. Kalau firewall, curl biasanya menggantung sampai timeout. `service nginx reload` hanya membuat Nginx yang **sudah hidup** membaca config baru, bukan menyalakannya.  
   > **Perbaikan:**
   > ```
   > nginx -t
   > service nginx start
   > ss -lntup | grep ':80'
   > ```
   > Kalau `service` tidak mau menyalakan, jalankan langsung dengan `nginx`. Kalau masih gagal, lihat `tail -n 20 /var/log/nginx/error.log`. Penyebab umumnya `Address already in use` (matikan dengan `pkill nginx`, lalu `nginx` lagi) atau error di config yang ditunjuk oleh `nginx -t`.

4. Pengujian di abbey
   ```
   curl -i -H "Host: abbey.k59.com" http://127.0.0.1/orion/
   curl -i -H "Host: static.k59.com" http://127.0.0.1/orion/
   ```
   Target: `HTTP/1.1 200 OK` dengan isi `<h1>Orion</h1>`.

   Buktikan juga bahwa PHP benar-benar tidak jalan:
   ```
   echo '<?php echo "x"; ?>' > /var/www/orion/tes.php
   curl -I -H "Host: static.k59.com" http://127.0.0.1/orion/tes.php
   rm /var/www/orion/tes.php
   ```
   Target: `HTTP/1.1 403 Forbidden`.

   Dari alpha:
   ```
   curl http://static.k59.com/orion/
   ```

   Direktori rendering  
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/cecd3800-be9a-4dcb-930f-219be24e87c1" /><br>
   Path dapat eksekusi file php di direktori yang /eternal
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/b47248e7-cfa9-4424-810b-23efb5f0dab9" /><br>
   Akses file PHP dari node lain  
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/2cf07685-de67-4e8d-bdcd-7b3816e5e874" /><br>
   Buat jalur /orion yang menyajikan directory /var/www/orion, secara murni statis tanpa perlu rendering PHP.
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/6c5268ac-7dd6-46f5-839a-81b915abf9ea" /><br>
   Isi dari web statisnya
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/09dcacc9-03e0-431d-babd-daa37b31006a" /><br>
   Testing eksekusi web statis sesuai jalur yang dibuat
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/c1f452f4-814f-4f56-a7fc-9059350acc91" /><br>
   
---

## Backup & autostart
Simpan perintah di script supaya mudah diulang kalau node di-restart: `/root/soal15.sh` di penny (langkah 1 sampai 6) dan di abbey (langkah 1 sampai 3). Supaya layanan menyala otomatis saat boot, tambahkan di network config masing-masing node, di bawah baris `up` yang sudah ada:

Penny:
```
up /usr/sbin/service php8.4-fpm start || true
```
Abbey:
```
up /usr/sbin/service nginx start || true
```

## Rangkuman error & perbaikan
| No | Gejala | Penyebab | Perbaikan |
| --- | --- | --- | --- |
| 1 | `curl: (7) Failed to connect to 127.0.0.1 port 80` di abbey | Nginx belum berjalan, tidak ada yang mendengarkan di port 80 | `nginx -t`, `service nginx start`, cek `ss -lntup \| grep ':80'` |
| 2 | (Temuan saat review config, belum diuji) vhost terpisah `eternal.conf` dengan `ServerName penny.k59.com` | Apache membaca file `sites-enabled` urut abjad, sehingga `eternal.conf` dibaca sebelum `penny-proxy.conf` dan bisa menimpa vhost proxy, termasuk redirect 301 soal 13 | `a2dissite eternal.conf`, lalu pindahkan `Alias /eternal` dan `ProxyPass /eternal !` ke dalam `penny-proxy.conf` |
| 3 | (Pencegahan) `/eternal` bisa terproxy ke vault dan menghasilkan 404 | `ProxyPass "/"` menangkap semua path, termasuk `/eternal` | Tulis `ProxyPass /eternal !` **sebelum** `ProxyPass "/"` |

## Pelajaran
* `connection refused` artinya tidak ada layanan yang mendengarkan di port itu. Periksa dulu apakah layanannya hidup (`ss -lntup`) sebelum menyalahkan config path.
* `reload` hanya untuk layanan yang sudah hidup. Kalau layanan mati, gunakan `start`.
* Pada reverse proxy, path yang ingin dilayani sendiri harus dikecualikan dari aturan proxy (`ProxyPass ... !` di Apache, `location` yang lebih spesifik di Nginx).
* Satu domain sebaiknya dilayani oleh satu vhost. Dua vhost dengan `ServerName` yang sama akan saling bertabrakan.

---

## Soal 17 (Revisi)
### Perintah soal
* Tambahkan **TXT record** pada DNS untuk semua klien sayap kiri dan sayap kanan (alpha, beta, gamma, delta, epsilon).
* Jika DNS di-query TXT terhadap nama domain mereka (contoh: `alpha.k59.com`), sistem harus mengembalikan teks berupa nama hostname mereka masing-masing (contoh: `"alpha"`).

### Konsep singkat
Kita sudah kenal **A record** (nama → IP). **TXT record** adalah catatan teks bebas yang ditempelkan ke sebuah nama. Isinya bukan IP dan bukan nama lain, hanya tulisan. Di buku telepon DNS, alpha sudah punya baris "alpha → 10.93.1.2". Sekarang kita tambah baris kedua untuk nama yang sama: "alpha → tulisan `alpha`". Satu nama boleh punya beberapa jenis record sekaligus, jadi record lama tidak diubah.

Alur kerjanya:
| Peran | Node | Tugas |
| --- | --- | --- |
| Master | prab (`10.93.3.2`) | Satu-satunya tempat zona boleh diedit |
| Slave | tedd (`10.93.3.3`) | Menyalin zona dari prab, **tidak boleh diedit manual** |

Tedd hanya menyalin kalau **nomor serial SOA naik**. Urutannya selalu: edit di prab → naikkan serial → reload → cek tedd.

> **PENTING:** langkah 1 sampai 5 dikerjakan di **prab**, langkah 6 di **tedd**, langkah 7 dan 8 untuk verifikasi. Cek prompt (`root@prab` atau `root@tedd`) sebelum mengetik.

### Langkah pengerjaan
1. Di prab, cek serial SOA sekarang dan record klien yang sudah ada
   ```
   grep -A1 'SOA' /etc/bind/k59/k59.com
   grep -E '^(alpha|beta|gamma|delta|epsilon)' /etc/bind/k59/k59.com
   ```
   Catat angka serialnya. Kelima klien harus muncul sebagai A record.

2. Naikkan serial SOA
   ```
   nano /etc/bind/k59/k59.com
   ```
   Naikkan angka serial satu tingkat (format `TahunBulanTanggalNomor`, misalnya `2026100101` menjadi `2026100102`). Tanpa kenaikan ini, tedd menganggap zonanya masih versi terbaru dan tidak menyalin.

3. Tambahkan TXT record di bagian bawah file zona
   ```
   cat >> /etc/bind/k59/k59.com <<'EOF'

   ; --- SOAL 17: TXT record klien ---
   alpha   IN      TXT     "alpha"
   beta    IN      TXT     "beta"
   gamma   IN      TXT     "gamma"
   delta   IN      TXT     "delta"
   epsilon IN      TXT     "epsilon"
   EOF
   ```
   * `>>` artinya **menambah** di akhir file. Kalau memakai `>`, seluruh isi zona (SOA, NS, A, CNAME) terhapus.
   * Tanda kutip `"..."` wajib untuk isi TXT. Itu penanda bahwa isinya teks.
   * Nama di kolom pertama (`alpha`) tanpa titik, jadi BIND menyambungnya otomatis menjadi `alpha.k59.com.`
   * Baris `EOF` harus **rata kiri** (tanpa spasi di depan), kalau tidak heredoc tidak berhenti.

4. Cek zona dan reload
   ```
   named-checkzone k59.com /etc/bind/k59/k59.com
   rndc reload
   ```
   Target: `loaded serial ...` dan `OK`. Kalau error, pesannya menunjuk nomor baris yang salah.

5. Tes di prab (master)
   ```
   for host in alpha beta gamma delta epsilon; do
       echo -n "$host: "
       dig @10.93.3.2 "$host.k59.com" TXT +short
   done
   ```
   Target:
   ```
   alpha: "alpha"
   beta: "beta"
   gamma: "gamma"
   delta: "delta"
   epsilon: "epsilon"
   ```
   <br><img width="546" height="158" alt="image" src="https://github.com/user-attachments/assets/20901953-00b0-4d12-9e03-7a5dd55422f4" /><br>

6. Pastikan tedd (slave) menyalin zona
   Di tedd, bandingkan serial prab dan tedd:
   ```
   dig @10.93.3.2 k59.com SOA +short
   dig @10.93.3.3 k59.com SOA +short
   ```
   Keduanya harus menampilkan serial yang sama. Kalau serial tedd kosong atau salah, lihat bagian error di bawah.

   > **Error 1 (BIND di tedd belum terpasang atau belum hidup):**
   > ```
   > root@tedd:~# dig @10.93.3.2 k59.com SOA +short
   > prab.k59.com. root.k59.com. 2026100101 604800 86400 2419200 604800
   > root@tedd:~# dig @10.93.3.3 k59.com SOA +short
   > ;; communications error to 10.93.3.3#53: connection refused
   > ;; no servers could be reached
   > root@tedd:~# rndc retransfer k59.com
   > bash: rndc: command not found
   > ```
   > **Penyebab:** DNS memakai port 53. `connection refused` artinya tidak ada layanan DNS yang mendengarkan di port itu di tedd. Petunjuk kedua adalah `rndc: command not found`. `rndc` ikut terinstal bersama paket BIND, jadi kalau `rndc` tidak ada, berarti BIND di tedd belum terpasang. Prab sehat (menjawab dengan serial `2026100101`), dan firewall bukan penyebab karena firewall biasanya membuat `dig` menggantung (timeout), bukan ditolak langsung.  
   > **Perbaikan (di tedd):**
   > ```
   > which named
   > ls -l /etc/init.d/ | grep -E 'bind|named'
   > apt-get update
   > DEBIAN_FRONTEND=noninteractive apt-get install -y bind9 bind9-utils dnsutils
   > ```
   > `bind9` adalah servernya, `bind9-utils` berisi `rndc` dan `named-checkconf`, dan `dnsutils` berisi `dig`.

   Setelah BIND terpasang, pasang konfigurasi slave. Pakai `nano` supaya aman dari masalah indentasi heredoc:
   ```
   nano /etc/bind/named.conf.options
   ```
   Isi dengan blok `options` saja:
   ```
   options {
       directory "/var/cache/bind";
       forwarders { 192.168.122.1; };
       dnssec-validation no;
       allow-query { any; };
       allow-recursion { any; };
       auth-nxdomain no;
       listen-on-v6 { any; };
   };
   ```
   ```
   nano /etc/bind/named.conf.local
   ```
   Isi dengan empat blok zona slave (zona utama dan tiga reverse zone dari soal 8):
   ```
   zone "k59.com" {
       type slave;
       masters { 10.93.3.2; };
       file "/var/lib/bind/k59.com";
   };

   zone "2.93.10.in-addr.arpa" {
       type slave;
       masters { 10.93.3.2; };
       file "/var/lib/bind/rev.2";
   };

   zone "3.93.10.in-addr.arpa" {
       type slave;
       masters { 10.93.3.2; };
       file "/var/lib/bind/rev.3";
   };

   zone "4.93.10.in-addr.arpa" {
       type slave;
       masters { 10.93.3.2; };
       file "/var/lib/bind/rev.4";
   };
   ```
   Blok `type slave` berarti tedd tidak punya file zona sendiri, dia menyalin dari `masters` (prab).

   > **Error 2 (zona terdefinisi dua kali):** saat `named-checkconf` dijalankan:
   > ```
   > root@tedd:~# named-checkconf
   > /etc/bind/named.conf.local:1: zone 'k59.com': already exists previous definition: /etc/bind/named.conf.options:11
   > /etc/bind/named.conf.local:4: writeable file '/var/lib/bind/k59.com': already in use: /etc/bind/named.conf.options:14
   > ... (diulang untuk tiga reverse zone)
   > ```
   > **Penyebab:** BIND menggabungkan beberapa file config, dan satu zona hanya boleh didefinisikan **sekali**. Pesan error menunjuk definisi pertama di `named.conf.options` baris 11 sampai 32, padahal file itu seharusnya hanya berisi blok `options` (sekitar 9 baris). Artinya blok zona ikut tertulis ke file `options`, dan juga ada di `named.conf.local`. Dugaan paling kuat: saat dua perintah `cat > ... <<'EOF'` dipaste sekaligus, `EOF` pertama tidak dikenali (misalnya karena menjorok akibat copy dari README), sehingga isi perintah berikutnya ikut tertulis ke file pertama. Dugaan ini belum diperiksa dengan `cat` pada kedua file.  
   > **Perbaikan:**
   > ```
   > cat /etc/bind/named.conf.options
   > cat /etc/bind/named.conf.local
   > ```
   > Buang blok `zone` dari `named.conf.options` sehingga isinya hanya blok `options { ... };` seperti di atas. Pastikan `grep -c '^zone' /etc/bind/named.conf.local` menghasilkan `4`. Lalu cek:
   > ```
   > named-checkconf
   > ```
   > **Tidak ada output = sukses.** Pada BIND, diam berarti beres. Perhatikan juga nama file di depan nomor baris pada pesan error (`named.conf.options` atau `named.conf.local`), karena itu menunjuk file yang bermasalah.

   Setelah `named-checkconf` bersih, nyalakan BIND:
   ```
   service named start
   service named status
   ss -lntup | grep ':53'
   sleep 3
   ls -l /var/lib/bind/
   ```
   Target: `bind is running`, port 53 mendengarkan, dan file `k59.com`, `rev.2`, `rev.3`, `rev.4` muncul di `/var/lib/bind/` (bukti transfer dari prab berhasil). Kalau `service` tidak mau, jalankan langsung dengan `/usr/sbin/named -u bind`.

7. Verifikasi TXT di tedd
   ```
   dig @10.93.3.2 k59.com SOA +short
   dig @10.93.3.3 k59.com SOA +short

   for host in alpha beta gamma delta epsilon; do
       echo -n "$host: "
       dig @10.93.3.3 "$host.k59.com" TXT +short
   done
   ```
   Target: dua baris SOA memiliki serial yang sama, dan kelima TXT muncul seperti di prab.

   Kalau SOA tedd tertinggal atau TXT kosong, paksa tedd menyalin ulang (sekarang `rndc` sudah ada):
   ```
   rndc retransfer k59.com
   ```

8. Verifikasi dari klien (misal alpha dan delta)
   ```
   dig alpha.k59.com TXT +short
   dig epsilon.k59.com TXT +short
   ```
   Target: `"alpha"` dan `"epsilon"`. Ini membuktikan hasilnya benar dari sisi pengguna, tidak hanya dari server DNS.

   Query TXT di prab & tedd  
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/bb5d5fdd-5d72-491f-ac53-9acd20e2b626" /><br>
   <br><img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/e2d90d93-70aa-4b2e-82ec-4a761bd35ebb" /><br>
   
   Soal 17 selesai.

### Rangkuman error & perbaikan
| No | Gejala | Penyebab | Perbaikan |
| --- | --- | --- | --- |
| 1 | `dig @10.93.3.3`: `connection refused`, dan `rndc: command not found` di tedd | BIND di tedd belum terpasang atau belum hidup, tidak ada layanan di port 53 | Install `bind9 bind9-utils dnsutils`, pasang config slave, `service named start`, cek `ss -lntup \| grep ':53'` |
| 2 | `named-checkconf`: `zone ... already exists previous definition` dan `writeable file ... already in use` | Blok zona ada di dua file (`named.conf.options` dan `named.conf.local`). Dugaan: heredoc `EOF` tidak dikenali saat paste | Rapikan `named.conf.options` agar hanya berisi blok `options`, lalu `named-checkconf` sampai tidak ada output |

### Pencegahan
| Kesalahan yang mungkin | Akibat |
| --- | --- |
| Lupa menaikkan serial SOA | `dig @10.93.3.2` benar, tetapi `dig @10.93.3.3` kosong karena tedd tidak menyalin |
| Memakai `>` bukan `>>` saat menambah TXT | Seluruh isi zona terhapus dan DNS rusak |
| Menulis isi TXT tanpa tanda kutip | BIND membaca kata sebagai token terpisah dan bisa error |
| Mengedit zona langsung di tedd | Editan hilang karena ditimpa saat tedd menyalin dari prab |
| Menimpa `named.conf.local` di prab hanya dengan zona `k59.com` (misalnya saat troubleshoot soal 12) | Reverse zone soal 8 bisa hilang. Cek dengan `cat /etc/bind/named.conf.local` (belum diperiksa) |

### Pelajaran
* `connection refused` pada `dig` artinya tidak ada layanan DNS yang mendengarkan di port 53. Periksa dulu apakah BIND terpasang dan hidup sebelum mencurigai isi zona.
* Kalau `rndc` tidak ditemukan, itu tanda paket BIND belum terpasang di node tersebut.
* Urutan aman menulis config BIND: **tulis → `named-checkconf` → baru start**. Kalau `named-checkconf` diam, lanjut.
* Zona hanya boleh didefinisikan sekali. Zona milik `named.conf.local`, dan `named.conf.options` hanya untuk blok `options`.
* Simpan langkah tedd sebagai script `/root/soal17-tedd.sh` dan pastikan network config tedd punya baris `up /usr/sbin/service named start || true`, karena BIND di tedd beberapa kali kembali kosong di soal 4, 17, dan sebelumnya.
