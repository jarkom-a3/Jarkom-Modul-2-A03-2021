# Jarkom-Modul-2-A03-2021

## Anggota

| Nama | NRP |
| :----: | :----: |
| Jessica Tasyanita | 05111940000043 |
| Daniel Sugianto | 05111940000075 |
| Ryan Fernaldy | 05111940000152 |

## Soal 1
EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet
- Konfigurasi Topologi GNS3
  - Foosha
    ```
    auto eth0
    iface eth0 inet dhcp

    auto eth1
    iface eth1 inet static
    address 192.170.1.1
    netmask 255.255.255.0

    auto eth2
    iface eth2 inet static
    address 192.170.2.1
    netmask 255.255.255.0
    ```
  - Loguetown
    ```
    auto eth0
    iface eth0 inet static
    address 192.170.1.2
    netmask 255.255.255.0
    gateway 192.170.1.1
    ```
  - Alabasta
    ```
    auto eth0
    iface eth0 inet static
    address 192.170.1.3
    netmask 255.255.255.0
    gateway 192.170.1.1
    ```
  - EniesLobby
    ```
    auto eth0
    iface eth0 inet static
    address 192.170.2.2
    netmask 255.255.255.0
    gateway 192.170.2.1
    ```
  - Water7
    ```
    auto eth0
    iface eth0 inet static
    address 192.170.2.3
    netmask 255.255.255.0
    gateway 192.170.2.1
    ```
  - Skypie
     ```
     auto eth0
     iface eth0 inet static
     address 192.170.2.4
     netmask 255.255.255.0
     gateway 192.170.2.1
     ```
- Konfigurasi Router (Foosha)<br>
  ```
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.170.0.0/16
  ```
  Kemudian jalankan ``cat /etc/resolv.conf``</br>
  ![image](https://user-images.githubusercontent.com/68326540/138800959-6502dc9c-04f8-45b7-ba04-732e39632c59.png)
  
- Konfigurasi DNS Server (EniesLobby dan Water7)<br>
  Memasukkan IP nameserver dari foosha ke /etc/resolv.conf
  ```
  echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
  
  Install dan jalankan aplikasi Bind9 sebagai DNS Server
  ```
  apt-get update
  apt-get install bind9 -y
  service bind9 start
  ```
- Konfigurasi Web Server (Skypie)<br>
  Memasukkan IP nameserver dari foosha ke /etc/resolv.conf
  ```
  echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
  
  Install dan jalankan Apache2 sebagai Web Server
  ```
  apt-get update
  apt-get install apache2 -y
  apt-get install php -y
  apt-get install libapache2-mod-php7.0 -y
  service apache2 start
  ```
- Konfigurasi Client (Loguetown dan Alabasta)<br>
  Memasukkan IP nameserver dari foosha ke /etc/resolv.conf
  ```
  echo nameserver 192.168.122.1 > /etc/resolv.conf
  ```
  
  Install dnsutils dan lynx (Web Browser)
  ```
  apt-get update
  apt-get install dnsutils -y
  apt-get install lynx -y
  ```
- Semua node dapat mengakses ke internet<br>
  ```ping google.com```<br>
  ![image](https://user-images.githubusercontent.com/68326540/139208695-46318b1e-38ee-4c75-ae65-afec22a1c64c.png)
 
### Kendala

* tidak ada

## Soal 2
Membuat domain website utama franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku di EniesLobby.
- EniesLobby
  - Konfigurasi domain franky.yyy.com pada /etc/bind/named.conf.local
    ```
    echo -e "zone \"franky.A03.com\" {
        type master;
        file \"/etc/bind/kaizoku/franky.A03.com\";
        allow-transfer { 192.170.2.3; };
    };" > /etc/bind/named.conf.local;
    ```
  - Membuat folder dan konfigurasi BIND data
    ```
    mkdir /etc/bind/kaizoku
    touch /etc/bind/kaizoku/franky.A03.com
    echo -e "
    ;
    ; BIND data file for local loopback interface
    ;
    \$TTL    604800
    @       IN      SOA     franky.A03.com. root.franky.A03.com. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      franky.A03.com.
    @       IN      A       192.170.2.4
    www     IN      CNAME   franky.A03.com.
    " > /etc/bind/kaizoku/franky.A03.com

    service bind9 restart
    ```
- Loguetown dan Alabasta
  ```
  echo nameserver 192.170.2.2 > /etc/resolv.conf
  ```
  ```ping franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138802933-5ca2db78-4329-40f2-bf3b-145931dee4ed.png) <br>
  ```ping www.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138803084-d516befa-c166-4b80-bb51-1c792c754483.png)

### Kendala

* tidak ada

## Soal 3
Membuat subdomain super.franky.yyy.com dengan alias `www.super.franky.yyy.com` yang mengarah ke Skypie
- EniesLobby
  Menambah konfigurasi BIND data 
  ```
  echo -e "
  super   IN  A   192.170.2.4
  www.super   IN  A 192.170.2.4
  " >> /etc/bind/kaizoku/franky.A03.com

  service bind9 restart
  ```
  
- Loguetown dan Alabasta<br>
  ```ping super.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138803428-99dc1cb4-006e-442d-8d8a-f3c4c475ca96.png) <br>
  ```ping www.super.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138803466-300b9e32-202e-4675-b095-5b12abda0eeb.png)

### Kendala

* tidak ada


## Soal 4
Membuat reverse domain untuk domain utama
- EniesLobby
  - Konfigurasi reverse domain pada /etc/bind/named.conf.local
    ```
    echo -e "
    zone \"2.170.192.in-addr.arpa\" {
        type master;
        file \"/etc/bind/kaizoku/2.170.192.in-addr.arpa\";
    };" >> /etc/bind/named.conf.local
    ```
  - Konfigurasi BIND data untuk reverse domain
  ```
  touch /etc/bind/kaizoku/2.170.192.in-addr.arpa
  echo -e "
  ;
  ; BIND data file for local loopback interface
  ;
  \$TTL    604800
  @       IN      SOA     franky.A03.com. root.franky.A03.com. (
                                2         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  2.170.192.in-addr.arpa.       IN      NS      franky.A03.com.
  4      IN      PTR       franky.A03.com.
  " >> /etc/bind/kaizoku/2.170.192.in-addr.arpa

  service bind9 restart
  ```
- Loguetown dan Alabasta
  ```
  host -t PTR 192.170.2.4
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139529932-8399d8e3-5f1a-4447-b303-90d3f2a569fd.png)

### Kendala

* Awalnya bingung menentukan IP untuk reverse DNS, tapi sudah bisa terselesaikan

## Soal 5
Membuat Water7 sebagai DNS Slave untuk domain utama
- EniesLobby <br>
  Konfigurasi domain franky.A03.com
  ```
  echo -e "zone \"franky.A03.com\" {
    type master;
    notify yes;
    also-notify { 192.170.2.3; } ;
    allow-transfer { 192.170.2.3; };
    file \"/etc/bind/kaizoku/franky.A03.com\";
    
  };" > /etc/bind/named.conf.local;


  echo -e "
  zone \"2.170.192.in-addr.arpa\" {
      type master;
      file \"/etc/bind/kaizoku/2.170.192.in-addr.arpa\";
  };" >> /etc/bind/named.conf.local

  service bind9 restart
  ```
- Water7 <br>
  Konfigurasi domain fraky.A03.com sebagai DNS Slave
  ```
  echo -e "zone \"franky.A03.com\" {
    type slave;
    masters { 192.170.2.2; };
    file \"/var/lib/bind/franky.A03.com\";
  };" > /etc/bind/named.conf.local;

  service bind9 restart
  ```
- Mematikan DNS Server EniesLobby<br>
  ```service bind9 stop```
- Loguetown dan Alabasta<br>
  ```
  Mengatur nameserver ke master dan slave
  echo -e "
  nameserver 192.170.2.2
  nameserver 192.170.2.3" > /etc/resolv.conf
  ```
  
  ```ping franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138804806-35433e80-ea44-4446-8980-bf846e36f720.png) <br>
  
  ```ping www.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138804827-87fc1da7-15ce-4aed-b098-d56279c6ff75.png)

### Kendala

* tidak ada

## Soal 6
Membuat subdomain mecha.franky.yyy.com dengan alias `www.mecha.franky.yyy.com` yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo
- EniesLobby <br>
  - Menambahkan konfigurasi BIND data pada /etc/bind/kaizoku/franky.A03.com
    ```
    echo -e "
    ns1   IN  A   192.170.2.3
    mecha   IN  NS ns1
    www     IN  CNAME   ns1
    " >> /etc/bind/kaizoku/franky.A03.com
    ```
  
  - Mengatur konfigurasi pada /etc/bind/named.conf.options
    ```
    echo -e "
    options {
            directory \"/var/cache/bind\";
            // dnssec-validation auto;
            allow-query{any;};

            auth-nxdomain no;    # conform to RFC1035
            listen-on-v6 { any; };
    };" > /etc/bind/named.conf.options
    ```
- Water7
  - Mengatur konfigurasi pada /etc/bind/named.conf.options
    ```
    echo -e "
    options {
            directory \"/var/cache/bind\";
            // dnssec-validation auto;
            allow-query{any;};

            auth-nxdomain no;    # conform to RFC1035
            listen-on-v6 { any; };
    };" > /etc/bind/named.conf.options
    ```
  
  - Konfigurasi subdomain yang didelegasikan ke Water7
    ```
    echo -e "zone \"mecha.franky.A03.com\" {
        type master;
        file \"/etc/bind/sunnygo/mecha.franky.A03.com\";
    };" > /etc/bind/named.conf.local;
    ```
  
  - Membuat BIND data untuk subdomain yang didelegasikan
    ```
    mkdir /etc/bind/sunnygo
    touch /etc/bind/sunnygo/mecha.franky.A03.com
    echo -e "
    ;
    ; BIND data file for local loopback interface
    ;
    \$TTL    604800
    @       IN      SOA     mecha.franky.A03.com. root.mecha.franky.A03.com. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      mecha.franky.A03.com.
    @       IN      A       192.170.2.4
    www     IN      CNAME   mecha.franky.A03.com.
    " > /etc/bind/sunnygo/mecha.franky.A03.com


    service bind9 restart

    ```
- Loguetown dan Alabasta<br>
  ```ping mecha.franky.A03.com```<br>
  ![image](https://user-images.githubusercontent.com/68326540/138806294-bb246103-8c4b-4ffe-b48f-3405373f15b1.png) <br>
  ```ping www.mecha.franky.A03.com```<br>
  ![image](https://user-images.githubusercontent.com/68326540/138806332-b37a5d22-de45-4664-a72f-8dccb4b086d7.png)

### Kendala

* tidak ada

## Soal 7
Membuat subdomain melalui Water7 dengan nama general.mecha.franky.yyy.com dengan alias `www.general.mecha.franky.yyy.com` yang mengarah ke Skypie
- Water7<br>
  Menambahkan konfigurasi BIND data di /etc/bind/sunnygo/mecha.franky.A03.com
  ```
  echo -e "
  general   IN  A   192.170.2.4
  www.general   IN  A 192.170.2.4
  " >> /etc/bind/sunnygo/mecha.franky.A03.com

  service bind9 restart
  ```
  
- Loguetown dan Alabasta<br>
  ```ping general.mecha.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138806581-d2dd255c-7215-4b43-9cab-ec6b8e67ed22.png)<br>
  ```ping www.general.mecha.franky.A03.com```
  ![image](https://user-images.githubusercontent.com/68326540/138806621-3a276c24-ca5d-4876-856a-40ee47e28c74.png)

### Kendala

* tidak ada

## Soal 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.yyy.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.yyy.com.
### Skypie<br>
  - Melakukan clone pada repository yang telah disediakan
  ```
  apt-get install git
  git config --global http.sslVerify false
  git clone https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom.git
  ```
  - Melakukan unzip ke semua file zip yang terdapat pada direktori hasil clone
  - Memindahkan hasil direktori unzip ke direktori /var/www
 ```
 cp -r Praktikum-Modul-2-Jarkom/franky /var/www/
 cp -r Praktikum-Modul-2-Jarkom/super.franky /var/www/
 cp -r Praktikum-Modul-2-Jarkom/general.mecha.franky /var/www/
 ```
  - Merename folder sesuai nama yang diminta
 ```
 mv /var/www/franky /var/www/franky.A03.com
 mv /var/www/super.franky /var/www/super.franky.A03.com
 mv /var/www/general.mecha.franky.A03.com /var/www/general.mecha.franky.A03.com
 ```
  - Mencopy /etc/apache2/sites-available/000-default.conf dengan tujuan /etc/apache2/sites-available/franky.A03.com.conf
  ```
  cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/franky.A03.com.conf
  ```
  - Mengubah file /etc/apache2/sites-available/franky.A03.com.conf
 ```
  nano /etc/apache2/sites-available/franky.A03.com.conf
 ```
  - Mengubah ServerName, ServerAlias, DocumentRoot 
  ```
  ServerName franky.A03.com
  ServerAlias www.franky.A03.com
  DocumentRoot /var/www/franky.A03.com
  ```
  ![image](https://user-images.githubusercontent.com/62937814/139183228-9128d126-ecf6-4048-9ecc-92ffd9291902.png)

  - Restart server dan mengaktifkan website url
  ```
  a2ensite franky.A03.com
  service apache2 restart
  ```
### Loguetown<br>
  - Connect server skypie
  ```
  echo "# nameserver 192.168.122.1
  nameserver 192.170.2.2
  nameserver 192.170.2.3" > /etc/resolv.conf
  ```
  - Mengakses www.franky.A03.com
  ```
  lynx http://www.franky.A03.com
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139530573-40da40e7-db24-427a-b6dc-928c8e086f0c.png)

### Kendala

* tidak ada  
  
## Soal 9
Setelah itu, Luffy juga membutuhkan agar url www.franky.yyy.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home. 
### Skypie<br>
  - Mengubah file /etc/apache2/sites-available/franky.A03.com.conf
 ```
  nano /etc/apache2/sites-available/franky.A03.com.conf
 ```
  - Menambahkan alias untuk url home 
  ```
  Alias "/home" "/var/www/super.franky.A03.com/index.php/home"
  ```
  ![image](https://user-images.githubusercontent.com/62937814/139183228-9128d126-ecf6-4048-9ecc-92ffd9291902.png)

### Loguetown<br>
  - Mengakses www.franky.A03.com/home
  ```
  lynx www.franky.A03.com/home
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139530600-345e5663-487d-4420-9209-73b90ed3d732.png)

### Kendala

* tidak ada
  
## Soal 10
Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com
### Skypie<br>
  - Menambahkan file conf untuk subdomain super /etc/apache2/sites-available/super.franky.A03.com.conf
 ```
  cp /etc/apache2/sites-available/franky.A03.com.conf /etc/apache2/sites-available/super.franky.A03.com.conf
  nano /etc/apache2/sites-available/super.franky.A03.com.conf
 ```
  - Mengubah ServerName, ServerAlias, DocumentRoot 
  ```
    ServerName super.franky.A03.com
    ServerAlias www.super.franky.A03.com
    DocumentRoot /var/www/super.franky.A03.com
  ```
### Loguetown<br>
  - Mengakses www.super.franky.A03.com
  ```
  lynx http://www.super.franky.A03.com
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139530611-239354cf-3b49-4ba0-ae1c-a452997ee44f.png)

### Kendala

* tidak ada  
## Soal 11
Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.
### Skypie<br>
  - Mengedit file /etc/apache2/sites-available/super.franky.A03.com.conf
 ```
  nano /etc/apache2/sites-available/franky.A03.com.conf
 ```
  - Menambahkan directory listing 
  ```
  <Directory /var/www/super.franky.A03.com>
      Options +Indexes
  </Directory>

  <Directory /var/www/super.franky.A03.com/public>
      Options +Indexes
  </Directory>
  ```
### Loguetown<br>
  - Mengakses www.super.franky.A03.com/public
  ```
  lynx http://www.super.franky.A03.com/public
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139530619-219e1399-fdc0-43af-a90a-5764fcbd7271.png)

### Kendala

* tidak ada

## Soal 12
Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache
### Skypie<br>
  - Mengedit file /etc/apache2/sites-available/super.franky.A03.com.conf
 ```
  nano /etc/apache2/sites-available/franky.A03.com.conf
 ```
  - Menambahkan eror page
  ```
  ErrorDocument 404 /error/404.html
  
  <Files "404.html">
      <If "-z %{ENV:REDIRECT_STATUS">
            RedirectMatch 404 ^/404.html$
      </If>
  </Files>
  ```
### Loguetown<br>
  - Mengakses www.super.franky.A03.com/abcdef
  ```
  lynx http://www.super.franky.A03.com/abcdef
  ```
  ![image](https://user-images.githubusercontent.com/68326540/139530640-1cbe2aa5-817a-43ce-a44d-06550467e77f.png)

### Kendala

* Bingung mencari syntax yang diperlukan

## Soal 13
Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js. 
### Skypie<br>
  - Mengedit file /etc/apache2/sites-available/super.franky.A03.com.conf
 ```
  nano /etc/apache2/sites-available/franky.A03.com.conf
 ```
  - Menambahkan konfigurasi alias
  ```
  <Directory /var/www/super.franky.A03.com/public/js>
      Options +Indexes
  </Directory>

  Alias "/js" "/var/www/super.franky.A03.com/public/js"
  ```

 ![image](https://user-images.githubusercontent.com/62937814/139184044-70bf9b73-cc28-44b4-aa7f-99ab7a366d49.png)
### Loguetown<br>
  - Mengakses lynx www.super.franky.A03.com/js <br>
    ![image](https://user-images.githubusercontent.com/68326540/139530741-9a4681b7-35b0-4825-8b27-ee6b13880ba2.png)

### Kendala

* tidak ada

## Soal 14
Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500

1. Pada node Skypie, buat file bernama `general.mecha.franky.A03.com.conf` dengan isi seperti di bawah. Dalam `<VirtualHost *:15000>` dan `<VirtualHost *:15500>` jadikan `general.mecha.franky.A03.com` sebagai `ServerName`, `www.general.mecha.franky.A03.com` sebagai `ServerAlias`, dan `/var/www/general.mecha.franky.A03.com` sebagai `DocumentRoot`.

```
<VirtualHost *:15000>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName general.mecha.franky.A03.com
        ServerAlias www.general.mecha.franky.A03.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.A03.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

<VirtualHost *:15500>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName general.mecha.franky.A03.com
        ServerAlias www.general.mecha.franky.A03.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.A03.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

2. Copy file di atas ke directory `/etc/apache2/sites-available/`.

```
cp general.mecha.franky.A03.com.conf /etc/apache2/sites-available/general.mecha.franky.A03.com.conf
```

3. Enable site `general.mecha.franky.A03.com`.

```
a2ensite general.mecha.franky.A03.com
```

4. Tambahkan port 15000 dan 15500 yang akan di-listen ke `/etc/apache2/ports.conf`

```
echo Listen 15000 >> /etc/apache2/ports.conf
echo Listen 15500 >> /etc/apache2/ports.conf
```

5. Restart apache2

```
service apache2 restart
```

6. Periksa apakah dapat mengakses melalui port 15000 dan port 15500

```
lynx www.general.mecha.franky.A03.com:15000
```

![15000](https://user-images.githubusercontent.com/58259649/139525735-9a42ea93-bc13-4ba6-a373-fa06be1aced7.png)

```
lynx www.general.mecha.franky.A03.com:15500
```

![15500](https://user-images.githubusercontent.com/58259649/139531024-cac8a8a8-f89b-492c-a4d9-3f698948fc5c.jpg)

### Kendala

* tidak ada

## Soal 15
dengan autentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy

1. Buat user baru dengan nama `luffy` dengan password `onepiece`.

```
htpasswd -c /etc/apache2/.htpasswd luffy
```

Isi dari `/etc/apache2/.htpasswd`

![luffypassword](https://user-images.githubusercontent.com/58259649/139531101-0eda1def-bca6-4635-9ea9-beeb69be81dd.jpg)

2. Tambahkan line-line di bawah ke dalam `<VirtualHost *:15000>` dan `<VirtualHost *:15500>` di `/etc/apache2/sites-available/general.mecha.franky.A03.com.conf`.

```
        <Directory "var/www/general.mecha.franky.A03.com">
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>
```

3. Coba akses `www.general.mecha.franky.A03.com:15000` atau `www.general.mecha.franky.A03.com:15500`.

![15500-login-user](https://user-images.githubusercontent.com/58259649/139531458-d07abf3d-58de-42aa-9545-5a3dfaa87ac8.jpg)

![15500-login-password](https://user-images.githubusercontent.com/58259649/139531462-f7e06e73-bc09-4a25-87e9-d91d500514e0.jpg)

![15500-login-success](https://user-images.githubusercontent.com/58259649/139531464-233641e5-29b8-471b-af0c-97dd3085db09.jpg)

## Soal 16
Dan setiap kali mengakses IP Skypie akan dialihkan secara otomatis ke www.franky.yyy.com

1. Pada node Skypie, buat file `.16htaccess` dengan isi sebagai berikut. `RewriteCond` menggunakan ip 192.170.2.4 (Skypie) dan `RewriteRule`

```
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^192\.170\.2\.4$
RewriteRule ^(.*)$ http://www.franky.A03.com/$1 [L,R=301]
```

2. Buat file `16.franky.A03.com.conf` dengan isi seperti di bawah. Gunakan `franky.A03.com` sebagai `ServerName`, `www.franky.A03.com` sebagai `ServerAlias`, `/var/www/franky.A03.com` sebagai `DocumentRoot` dan `Alias "/home" "/var/www/franky.A03.com/index.php/home"`. Dalam `<Directory /var/www/franky.A03.com>` tambahkan `Options +FollowSymLinks -Multiviews` dan `AllowOverride All`.

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        ServerName franky.A03.com
        ServerAlias www.franky.A03.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.A03.com
        Alias "/home" "/var/www/franky.A03.com/index.php/home"

        <Directory /var/www/franky.A03.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

3. Buat file `000-default.conf` dengan isi sebagai berikut. Gunakan `www.franky.A03.com` sebagai `ServerName` dan `/var/www/franky.A03.com` sebagai `DocumentRoot`.

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.


        ServerName www.franky.A03.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.A03.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

4. Enable modul mod_rewrite untuk Apache.

```
a2enmod rewrite
```

5. Restart Apache.

```
service apache2 restart
```

6. Copy `.16htaccess` ke `/var/www/franky.A03.com/` dengan nama `.htaccess`.

```
cp .16htaccess /var/www/franky.A03.com/.htaccess
```

7. Copy `16.franky.A03.com.conf` ke `/etc/apache2/sites-available/` dengan nama `franky.A03.com.conf`.

```
cp 16.franky.A03.com.conf /etc/apache2/sites-available/franky.A03.com.conf
```

8. Ubah `/etc/apache2/sites-available/000-default.conf` dengan `000-default.conf` yang terbaru.

```
a2dissite 000-default.conf
service apache2 restart
cp 000-default.conf /etc/apache2/sites-available/000-default.conf
a2ensite 000-default.conf
service apache2 restart
```

9. Coba akses IP Skypie.

```
lynx 192.170.2.4
```

![16](https://user-images.githubusercontent.com/58259649/139526490-fabce794-b0d1-43a1-b9ba-cfa84f897616.png)

### Kendala

* tidak ada

## Soal 17
Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring “franky” akan diarahkan menuju franky.png. Maka bantulah Luffy untuk membuat konfigurasi dns dan web server ini!
