> From acudovs - [powergslb](https://github.com/acudovs/powergslb).


# PowerGSLB
PowerGSLB adalah service Global Server Load Balancing (GSLB). Global Server Load Balancing (GSLB) sendiri berguna untuk menentukan trafik dengan menggunakan service DNS. Berbeda dengan load balancing menggunakan reverse proxy, GSLB menentukan trafik dari dua server berdasarkan IP dan domain.

PowerGSLB merupakan salah satu service pendukung dari PowerDNS yang berfungsi sebagai GSLB. PowerGSLB ini dapat diinstal pada sistem operasi CentOS 7.

## Instalasi
### Instal dan Setup PowerGSLB dan PowerDNS
Siapkan dan atur manajer paket serta instal paket yang dibutuhkan nantinya:
```
yum -y install epel-release
yum -y update
yum -y install python2-pip
pip install pyping
```
Instal PowerGSLB dengan paket penginstalan dari github yang sudah termasuk PowerDNS didalamnya:
```
VERSION=1.7.4
yum -y --setopt=tsflags= install \
    "https://github.com/AlekseyChudov/powergslb/releases/download/$VERSION/powergslb-$VERSION-1.el7.noarch.rpm" \
    "https://github.com/AlekseyChudov/powergslb/releases/download/$VERSION/powergslb-admin-$VERSION-1.el7.noarch.rpm" \
    "https://github.com/AlekseyChudov/powergslb/releases/download/$VERSION/powergslb-pdns-$VERSION-1.el7.noarch.rpm"
```
Konfigurasi dan masukkan password database yang akan digunakan PowerGSLB:
```
sed -i 's/^password = .*/password = your-database-password-here/g' /etc/powergslb/powergslb.conf
```
Copy file konfigurasi PowerDNS dari paket penginstalan ke dalam direktori etc:
```
cp /etc/pdns/pdns.conf /etc/pdns/pdns.conf~
cp "/usr/share/doc/powergslb-pdns-$VERSION/pdns/pdns.conf" /etc/pdns/pdns.conf
```

### Instal dan Setup MariaDB
Lakukan instalasi MariaDB:
```
yum -y install mariadb-server
```
Konfigurasi bind address MariaDB:
```
sed -i '/\[mysqld\]/a bind-address=127.0.0.1\ncharacter_set_server=utf8' /etc/my.cnf.d/server.cnf
```
Jalankan service MariaDB dan lihat statusnya:
```
systemctl enable mariadb.service
systemctl start mariadb.service
systemctl status mariadb.service
```
Jalankan konfigurasi keamanan mysql jika diperlukan: (Optional)
```
mysql_secure_installation
```
Buat user dan database beserta strukturnya menggunakan file sql yang ada di dalam paket penginstalan:
```
VERSION=1.7.4
mysql -p << EOF
CREATE DATABASE powergslb;
GRANT ALL ON powergslb.* TO powergslb@localhost IDENTIFIED BY 'your-database-password-here';
USE powergslb;
source /usr/share/doc/powergslb-$VERSION/database/scheme.sql
source /usr/share/doc/powergslb-$VERSION/database/data.sql
EOF
```
Jalankan service PowerGSLB dan PowerDNS:
```
systemctl enable powergslb.service pdns.service
systemctl start powergslb.service pdns.service
systemctl status powergslb.service pdns.service
```
Izinkan port HTTP, HTTPS, dan DNS:
```
firewall-cmd --permanent --add-port=80/tcp --permanent
firewall-cmd --permanent --add-port=443/tcp --permanent
firewall-cmd --permanent --add-port=53/tcp --permanent

firewall-cmd --permanent --add-service=http --permanent
firewall-cmd --permanent --add-service=https --permanent
firewall-cmd --permanent --add-service=dns --permanent

firewall-cmd --reload
firewall-cmd --list-all
```
