# LEMP Dokumantasyonu

### İçindekiler
- [Giriş](#giriş)
- [Kurulum](#kurulum)
- [Yapılandırma](#yapılandırma)
  - [Nginx Yapılandırması](#nginx)
  - [MySQL Yapılandırması ](#mysql)
  - [PHP(Wordpress) Yapılandırması ](#php)
  - [Windowsda Domain Yapılandırması ](#windows) 
- [Yönetim](#yönetim)


## Giriş
LEMP yığını , Linux İşletim sistemi üzerinde çalışan NGİNX ,MySQL ve PHP bileşenlerini 
içerir. Bu dökümantasyonda LEMP uygulamasının kurulum, yapılandırma ve yönetimi 
hakkında bilgi sağlanacaktır.

## Kurulum
  
  -  Linux İşletim Sistemi
 Bir Linux işletim sistemi dağıtımı olan Ubuntu kuruldu.
 -  Nginx Kurulumu
 Paket yöneticisi aracılığıyla Nginx kuruldu.
 -  MySQL Kurulumu
 Paket yöneticisi aracılığıyla MySQL kuruldu.
 -  PHP Kurulumu
 Paket yöneticisi aracılığıyla php alt yapısıyla hazırlanmış wordpress hazır tema kuruldu

 ## Yapılandırma

  ### Nginx

  /etc/nginx/sites-available dizininde konfigürasyon yapılmak için wordpress adında bir 
dosya oluşturuldu ve bu dosya /etc/nginx/sites-enabled dizini ile linklendi.

 #### - Wordpress projesinin Localhost olarak çalışması için yapılan nginx konfigürasyonları ;

`İki tane site olduğu için bu konfigürasyonlar /etc/nginx/sites-available dizinin içindeki site1 ve site2 dosyalarının her ikisi içinde yapılmıştır.`

Bir server bloğu içinde

 ```bash
    Server {
    
        listen 80;
        listen [::]:80;
    } 
   ```
- Domain belirtildi ;
   ```bash
     server_name (site1 yada site2);
   ```
- Wordpress projesinin kodlarının yolu belirtildi ;
   ```bash
     root /var/www/(site1 yada site2)/wordpress;
   ```
 - İndex olarak gösterilmesi gereken dosya belirtildi;
   ```bash
    index index.php index.html index.htm;
   ```
 - Son olarak PHP dosyalarının işlenmesi için gerekli ayarlamalar yapıldı;
  
   ```bash
    location / {
    
            try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
    
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
    }
    ```
- Ve bu ayarlamalar yapıldıktan sonra nginx yeniden başlatıldı;
  
  ```bash
    service nginx restart
  ```   


### MySQL

`İki tane site için ayrı ayrı database ve user oluşturulmuştur`

 - Yüklenen mysql içinde wordpress uygulaması ile bağlantı kurmak üzere wordpress adında bir veritabanı oluşturuldu;
    ```bash 
    CREATE DATABASE wordpress;
    ``` 
 - Veri tabanına bağlanabilecek bir user oluşturuldu;
    ```bash
    CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'parola';
    ``` 
- Oluşturulan usera veritabanına erişim izni verildi;
    ```bash
    GRANT ALL PRIVILEGES ON wordpress.* TO 'kullanici_adi'@'localhost';
    ``` 

### PHP

- `/var/www/(site1 yada site2)/wordpress` dizininde yer alan `wp-config.php` dosyasında veritabanı bağlantısı için 
 gereken bilgiler eklendi.

###    Windows
- ``C:\Windows\System32\drivers\etc\hosts`` dizininde yer alan hosts     dosyasını açıp gerekli ayarlamalar yapıldı;

    ``` bash
        192.168.1.104 site1
        192.168.1.104 site2
    ```


 ## Yönetim

 ### MySQL Backup

- `/backups/sql` dizinine eklenmek üzere her gün çalışacak bir mysqldump komutunu 
 kullanan backup scripti yazıldı.
 
    `/scripts/(site1 veya site2)-sql-backup-scrpt.sh` dizinindeki script;
 
    ``` bash
        #!/bin/bash
        NOW=$(date +"%d.%m.%Y")
        # Mysql
        BACKUP_FILE="/backups/sql/mysql-$NOW.sql"
        mysqldump -u root -h localhost -p wordpress > $BACKUP_FILE
    ```
- Bu scriptin hergün belli bir saatte otomatik olarak çalışması için crontab kullanıldı;
 
   Crontab ayarı = 
    ``` bash
        30 2 * * * /scripts/sql-backup-scrpt.sh 
    ```
### Wordpress Backup

- Tar komutu kullanılarak wordpress dosyaları sıkıştırılıp yedeklendi;
   
   ```bash
     tar cvzf /backups/wp-backup.tar.gz /var/www/(site1 yada site2)*
   ```
    

