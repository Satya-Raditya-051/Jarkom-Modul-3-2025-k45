# Jarkom-Modul-3-2025-k45

| Nama                            | NRP        |
| ------------------------------- | ---------- |
| I Dewa Made Satya Raditya       | 5027231051 |
| Made Gede Khrisna Wangsa        | 5027201047 | 


## No.1
**SOAL:** 
Di awal Zaman Kedua, setelah kehancuran Beleriand, para Valar menugaskan untuk membangun kembali jaringan komunikasi antar kerajaan. Para Valar menyalakan Minastir, Aldarion, Erendis, Amdir, Palantir, Narvi, Elros, Pharazon, Elendil, Isildur, Anarion, Galadriel, Celeborn, Oropher, Miriel, Amandil, Gilgalad, Celebrimbor, Khamul, dan pastikan setiap node (selain Durin sang penghubung antar dunia) dapat sementara berkomunikasi dengan Valinor/Internet (nameserver 192.168.122.1) untuk menerima instruksi awal.

**PENGERJAAN:** 
buat topografinya seperti ini
<img width="755" height="581" alt="Screenshot 2025-11-02 023558" src="https://github.com/user-attachments/assets/c1c9b529-e8d8-4203-9b6a-f89696e7ed4d" />

tambahkan setup router (Durin)
```
# DHCP config for eth0
auto eth0
iface eth0 inet dhcp
#	hostname ervn-debi-new-1

# Static config for eth1
auto eth1
iface eth1 inet static
	address 10.86.1.1
	netmask 255.255.255.0

# Static config for eth2
auto eth2
iface eth2 inet static
	address 10.86.2.1
        netmask 255.255.255.0

# Static config for eth3
auto eth3
iface eth3 inet static
	address 10.86.3.1
	netmask 255.255.255.0

# Static config for eth4
auto eth4
iface eth4 inet static
	address 10.86.4.1
	netmask 255.255.255.0

# Static config for eth5
auto eth5
iface eth5 inet static
	address 10.86.5.1
	netmask 255.255.255.0

up echo nameserver 192.168.122.1 > /etc/resolv.conf
up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.86.0.0/16
```

setup config di node static (sesuaikan lagi dengan eth durin)
```
# Static config for eth0
auto eth0
iface eth0 inet static
	address 10.86.1.2 (sesuaikan eth yg terhubung)
	netmask 255.255.255.0
	gateway 10.86.1.1 (sesuaikan durin)
	up echo nameserver 192.168.122.1 > /etc/resolv.conf
```


## No.2
**SOAL:**
Raja Pelaut Aldarion, penguasa wilayah Númenor, memutuskan cara pembagian tanah client secara dinamis. Ia menetapkan:
- Client Dinamis Keluarga Manusia: Mendapatkan tanah di rentang [prefix ip].1.6 - [prefix ip].1.34 dan [prefix ip].1.68 - [prefix ip].1.94.
- Client Dinamis Keluarga Peri: Mendapatkan tanah di rentang [prefix ip].2.35 - [prefix ip].2.67 dan [prefix ip].2.96 - [prefix ip].2.121.
- Khamul yang misterius: Diberikan tanah tetap di [prefix ip].3.95, agar keberadaannya selalu diketahui. Pastikan Durin dapat menyampaikan dekrit ini ke semua wilayah yang terhubung dengannya.

**PENGERJAAN:**
1. Aldarion:
   - install paket dhcp server
     ```
      apt-get update
      apt-get install isc-dhcp-server -y
     ```
   - konfig `nano /etc/default/isc-dhcp-server` dan `tambahkan INTERFACESv4="eth0"`
   - konfig `nano /etc/dhcp/dhcpd.conf` dan tambahkan:
     ```
     # declare sebagai server resmi di jaringan ini
      authoritative;
     
      # Kita arahkan DNS ke Erendis (10.86.3.3) dan 192.168.122.1 (kalo semisal e erendis gabisa)
      option domain-name-servers 10.86.3.3, 192.168.122.1; 
      default-lease-time 600;
      max-lease-time 7200;

      # Subnet 1: Keluarga Manusia (untuk Amandil)
      subnet 10.86.1.0 netmask 255.255.255.0 {
      option routers 10.86.1.1;
      option subnet-mask 255.255.255.0;
      option broadcast-address 10.86.1.255;
    
      # Rentang IP [prefix].1.6 - [prefix].1.34 
      range 10.86.1.6 10.86.1.34;
      # Rentang IP [prefix].1.68 - [prefix].1.94
      range 10.86.1.68 10.86.1.94;
      }

      # Subnet 2: Keluarga Peri (untuk Gilgalad)
      subnet 10.86.2.0 netmask 255.255.255.0 {
      option routers 10.86.2.1;
      option subnet-mask 255.255.255.0;
      option broadcast-address 10.86.2.255;
    
      # Rentang IP [prefix].2.35 - [prefix].2.67 
      range 10.86.2.35 10.86.2.67;
      # Rentang IP [prefix].2.96 - [prefix].2.121 
      range 10.86.2.96 10.86.2.121;
      }

      # Subnet 3: Wilayah Khamul
      subnet 10.86.3.0 netmask 255.255.255.0 {
      option routers 10.86.3.1;
      option subnet-mask 255.255.255.0;
      option broadcast-address 10.86.3.255;
    
      # IP Tetap untuk Khamul [prefix].3.95 [cite: 127]
      host Khamul {
        hardware ethernet 02:42:ab:05:83:00; 
        fixed-address 10.86.3.95;
        }
      }

      # Subnet 4: Jaringan Server (Aldarion)
      subnet 10.86.4.0 netmask 255.255.255.0 {
      option routers 10.86.4.1;
     }
     ```
   - restart dhcp server `service isc-dhcp-server restart`
     
2. Durin:
   - install dhcp relay
      ```
      apt-get update
      apt-get install isc-dhcp-relay -y
      ```
   - konfig relay di `nano /etc/default/isc-dhcp-relay`
     ```
     # IP DHCP Server (aldarion)
      SERVERS="10.86.4.4"

      # Interface yang "mendengarkan" request DHCP dari client
      INTERFACES="eth1 eth2 eth3 eth4"
     ```
   - konfig ip forwarding dengan `nano /etc/sysctl.conf` dan tambahkan `net.ipv4.ip_forward=1`
   - cek status ip forwarding dengan `sysctl -p`
   - restart dhcp relay dengan `service isc-dhcp-relay restart`
  
**SOAL 3:**
Untuk mengontrol arus informasi ke dunia luar (Valinor/Internet), sebuah menara 
pengawas, Minastir didirikan. Minastir mengatur agar semua node (kecuali Durin) 
hanya dapat mengirim pesan ke luar Arda setelah melewati pemeriksaan di Minastir. 

**PENGERJAAN:**
1. Di minastir:
   - Intall dnsmasq
     ```
     apt-get update
	 apt-get install dnsmasq -y
     ```
   - masuk ke konfig `nano /etc/dnsmasq.conf` dan tambahkan
     ```
	 # agar dnsmasq untuk membaca /etc/resolv.conf untuk upstream
		no-resolv
     
	 # upstream server DNS "Valinor"/Internet, Hanya Minastir yang akan berbicara dengan alamat ini.
		server=192.168.122.1

	 # Memberitahu dnsmasq untuk mendengarkan di IP-nya sendiri
		listen-address=127.0.0.1, 10.86.5.2
     ```
   - atur resolver minastir dengan `echo "nameserver 127.0.0.1" > /etc/resolv.conf`
   - restart dnsmasq dengan `service dnsmasq restart`
     
2. Arahkan node lain ke minastir:
   node dinamis:
	- masuk ke aldarion
 	- edit konfig dhcpd.conf: `nano /etc/dhcp/dhcpd.conf`
    - di baris `option domain-name-servers ...`, ubah menjadi `option domain-name-servers 10.86.5.2;` untuk mengarahkan ke mianstir
    - restart dhcp servernya aldarion dengan `service isc-dhcp-server restart`
    - restart client dinamis lainnya (Amandil, Gilgalad, Khamul) dengan stop & start
   node statis:
	- masuk ke setiap node statis dan ganti nameserver dengan command `echo "nameserver 10.86.5.2" > /etc/resolv.conf` atau masukkan dalam config

3. Di durin:
    - masukan command iptables berikut:
      ```
		# 1. Izinkan minastir (10.86.5.2) mengirim query DNS (port 53) ke Internet
		iptables -A FORWARD -s 10.86.5.2 -o eth0 -p udp --dport 53 -j ACCEPT
		iptables -A FORWARD -s 10.86.5.2 -o eth0 -p tcp --dport 53 -j ACCEPT

		# 2. Blokir semua node lain dari mengirim query DNS ke Internet
		# blokir berdasarkan interface sumber (eth1, eth2, eth3, eth4)
		iptables -A FORWARD -i eth1 -o eth0 -p udp --dport 53 -j DROP
		iptables -A FORWARD -i eth1 -o eth0 -p tcp --dport 53 -j DROP

		iptables -A FORWARD -i eth2 -o eth0 -p udp --dport 53 -j DROP
		iptables -A FORWARD -i eth2 -o eth0 -p tcp --dport 53 -j DROP

		iptables -A FORWARD -i eth3 -o eth0 -p udp --dport 53 -j DROP
		iptables -A FORWARD -i eth3 -o eth0 -p tcp --dport 53 -j DROP

		iptables -A FORWARD -i eth4 -o eth0 -p udp --dport 53 -j DROP
		iptables -A FORWARD -i eth4 -o eth0 -p tcp --dport 53 -j DROP
      ```
4. Verifikasi:
	- masuk ke salah satu client statis
	- cek `cat /etc/resolv.conf` pastikan isinya nameserver 10.86.5.2
 	- coba ping google.com. Jika berhasil, Minastir bekerja

**SOAL 4:**
Ratu Erendis, sang pembuat peta, menetapkan nama resmi untuk wilayah utama (<xxxx>.com). Ia menunjuk dirinya (ns1.<xxxx>.com) dan muridnya Amdir (ns2.<xxxx>.com) sebagai penjaga peta resmi. Setiap lokasi penting (Palantir, Elros, Pharazon, Elendil, Isildur, Anarion, Galadriel, Celeborn, Oropher) diberikan nama domain unik yang menunjuk ke lokasi fisik tanah mereka. Pastikan Amdir selalu menyalin peta (master-slave) dari Erendis dengan setia.

**PENGERJAAN:**
1. Erendis:
   - install bind9
     ```
	 apt-get update
	 apt-get install bind9 -y
     ```
   - konfig named.conf.options agar Erendis meneruskan query yang tidak ia ketahui ke minastir lewat `nano /etc/bind/named.conf.options`.
     ```
		options {
       		directory "/var/cache/bind";

    	 forwarders {
        10.86.5.2; // IP Minastir
    	};

     	allow-query { any; };
		};
     ```
   - konfig named.conf.local untuk menandakan erendis sebagai master lewat `nano /etc/bind/named.conf.local`
     ```
		zone "k45.com" {
    	type master;
    	file "/etc/bind/db.k45.com"; 
    	allow-transfer { 10.86.3.2; };
		};
     ```
   - buat file zone `nano /etc/bind/db.k45.com`
     ```
     	;
		$TTL    604800
		@       IN      SOA     ns1.k45.com. root.k45.com. (
   		                           1         ; Serial 
                              604800         ; Refresh
                               86400         ; Retry
                             2419200         ; Expire
                              604800 )       ; Negative Cache TTL
		;
		 
		@       IN      NS      ns1.k45.com.
		@       IN      NS      ns2.k45.com.

		
		ns1     IN      A       10.86.3.3       ; IP Erendis
		ns2     IN      A       10.86.3.2       ; IP Amdir

		; A record lokasi penting  
		palantir  IN    A       10.86.4.3
		elros     IN    A       10.86.1.6
		pharazon  IN    A       10.86.2.6
		elendil   IN    A       10.86.1.2
		isildur   IN    A       10.86.1.3
		anarion   IN    A       10.86.1.4
		galadriel IN    A       10.86.2.2
		celeborn  IN    A       10.86.2.3
		oropher   IN    A       10.86.2.4
     ```
 	- cek config zone `named-checkzone k45.com /etc/bind/db.k45.com` sampai muncul `ok`
 	- restart bind9 `service named restart`

2. Amdir:
   - install bind9
     ```
		apt-get update
		apt-get install bind9 -y
     ```
   - konfig named.conf.options
     ```
		options {
    		forwarders {
        		10.86.5.2; // IP Minastir
    	   };
    	allow-query { any; };
	  	};
     ```
    - konfig named.conf.local untuk deklarasi amdir sebagai slave
      ```
		zone "k45.com" {
    		type slave;
    		file "/var/cache/bind/db.k45.com"; 
    		masters { 10.86.3.3; }; ]
			};
      ```
     - cek error `named-checkconf`
     - restart bind9 `service bind9 restart`
       
3. Minastir:
   - edit konfig dnsmasq `nano /etc/dnsmasq.conf` untuk forward semua query untuk k45.com ke Erendis, tambakan diatas setup sebelumnya
     ```
     	server=/k45.com/10.86.3.3
     ```
   - restart dnsmasq `service dnsmasq restart`

4. cek dari klien:
   - masuk ke klien manapun
   - masukkan `dig (uniquename).k45.com` cek output memberikan ip yang sesuai atau tidak

**SOAL 5:**
Untuk memudahkan, nama alias www.<xxxx>.com dibuat untuk peta utama <xxxx>.com. Reverse PTR juga dibuat agar lokasi Erendis dan Amdir dapat dilacak dari alamat fisik tanahnya. Erendis juga menambahkan pesan rahasia (TXT record) pada petanya: "Cincin Sauron" yang menunjuk ke lokasi Elros, dan "Aliansi Terakhir" yang menunjuk ke lokasi Pharazon. Pastikan Amdir juga mengetahui pesan rahasia ini.

**PENGERJAAN:**

