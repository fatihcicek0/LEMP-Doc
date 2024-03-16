# Disk ekleme

## Herşeyden önce bir disk ekliyoruz ben VirtualBox da sanal makine kullandığım için;

- storage -> Controller :SATA -> Add Hard Disk 

yolunu izledim.



#### tabloyu görmek için ;
  ````bash
   lsblk
  ````


### 1.adım ;
 fdisk komutu ile eklediğimiz diskin dizinini yazıyoruz

````bash
 sudo fdisk /dev/sdb
````
### 2.adım ;
 fdisk komutlarını öğrenmek için çıkan ekranda m ye basıyoruz

````bash
 m
````
### 3.adım ;
 “g” komutu, bir GUID bölüm tablosu (GPT) oluşturmak için kullanılır. GPT, fiziksel sabit disklerdeki bölüm tablosunun düzeni için bir standarttır.

````bash
 g
````
### 4.adım ;
yeni bir partition eklemek 

````bash
 n
````
### 5.adım ;
partition number seçmek ; deault olarak bir veriyor ama değiştirebilirsiniz. 

````bash
 opsiyonel olarak istediğiniz 1-128 arası partition numberi yazın. ya da enter layın
````
### 6.adım ;
partition botuyunu seçmek  ; ne kadar alana ihtiyacınız varsa yazıyorsunuz örneğin 10GB için;
````bash
 +10G
````
### 7.adım ;
İşlemi kaydedip bitirmek için;
````bash
 w
````

### Bu işlemleri yaptıktan sonra Mount işlemi yapmamız gerekir;

# Mount işlemi Nedir ?
Ubuntu işletim sisteminde “mounted on”, bir dosya sistemi veya disk bölümünün belirli bir noktaya bağlanması anlamına gelir. Bu, dosya sisteminin kullanılabilir hale getirilmesi için yapılan bir işlemdir.

Örneğin, bir harici sabit diskiniz varsa ve bunu Ubuntu’da kullanmak istiyorsanız, bu diski “mount” etmeniz gerekir. Bu, diskin Ubuntu dosya sistemi hiyerarşisine bağlanmasını sağlar. Böylece, diske erişebilir ve üzerinde dosyalar oluşturabilir veya okuyabilirsiniz.

“Mounted on” ifadesi, df komutu ile kullanıldığında, mevcut dosya sistemlerinin hangi noktalara bağlı olduğunu gösterir. Örneğin:

```bash
    $ df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda1        20G   10G   10G  50% /
    /dev/sdb1       100G   50G   50G  50% /mnt/mydisk
```
Yukarıdaki çıktıda, /dev/sda1 dosya sistemi “/” kök dizinine bağlıdır ve /dev/sdb1 dosya sistemi “/mnt/mydisk” noktasına bağlıdır. Bu, /mnt/mydisk dizinindeki dosyaların harici diske ait olduğunu gösterir.

Kısacası, “mounted on”, dosya sistemlerinin belirli bir noktaya bağlanmasını ifade eder ve bu işlem, dosya sistemlerini kullanılabilir hale getirir.

## Mount işlemi için ;

### 1.Adım 
 Öncelikle File sistemi Tanıtıyoruz;  
  -   ext4 için

```bash
 sudo mkfs.ext4 /dev/sdb1 
```
### 2.Adım 

 Bir dizin oluşturup ve diski bu dizine bağlıyoruz:  

```bash
 sudo mkdir /mnt/mydisk
 sudo mount /dev/sdb1 /mnt/mydisk 
```
# Swap Alanı

  - Swap (Takas) Alanı, sabit disk üzerinde işletim sistemi tarafından ayrılmış bir bölümdür. İşlenecek veriler RAM’e sığmadığı zaman bu bölüm RAM gibi kullanılır ve böylece işlemlerin devam etmesi sağlanır. Swap alanı, RAM miktarı ihtiyacı karşılamadığı durumlarda kullanmak için işletim sisteminin geçici olarak kullandığı bir alandır. Kısacası, bu alan size kullanmak için daha büyük RAM kapasitesi sağlamaktadır.
  - Swap alanı, sabit disk üzerinde oluşturulur.
  - Sabit disklerin veri okuma/yazma hızları RAM’den daha düşüktür, bu nedenle swap alanının kullanılması işlemleri yavaşlatır.
  - Swap alanı, veri kaybı riski olmadan kullanılabilir.

## Swap Alanı Ekleme

### 1. Adım 
####  Öncelikle swap ve Ram kontrolü yapıyoruz.  
```bash
 free --mega -t 
```
### 2. Adım 
####  Swap dosyasının boyutunu hesaplama.  
```bash
 1024*1024*[GB cinsinden istediğiniz alan]
```

### 3. Adım 

### Dosyayı Oluşturma
Aşağıdaki komut 3 GB boyutunda, (girdi dosyası /dev/zero olduğu için) 0 bitleri ile dolu /swap_file/swap_file1 dosyasını oluşturur.
```bash
 dd if=/dev/zero of=/swap_file/swap_file1 bs=1024 count=3145728
```

### 4.Adım
Dosya izinlerini Oluşturma;
```bash
 chmod -R 600 /swap_file
```
### 5.Adım
Dosyayı swap haline getirme;
```bash
 mkswap /swap_file/swap_file1
```

### 6.Adım
Swap Dosyasını Tek Seferlik Aktif Etme
```bash
 swapon /swap_file/swap_file1
```

### 7.Adım

Swap Dosyasını Fstab Dosyasına Kaydetme;
- Dosya sistemi tablosuna (fstab) bu dosyayı eklemeliyiz ki bilgisayarı açtığınızda otomatik olarak swap aktif edilsin.
```bash
echo "/swap_file/swap_file1     swap     swap    defaults    0 0" >> /etc/fstab
```
- Bu komut yerine bir metin düzenleme aracı ile de aşağıdaki metni /etc/fstab dosyasına yapıştırabilirsiniz.
```bash
    /swapfile     swap     swap    defaults    0 0
```

# Tüm Swap Dosyalarını Aktif/Deaktif Etme

- Aşağıdaki komut ile tüm swap dosyalarını deaktif edebilirsiniz.
```bash
 swapoff -a
```
- Aşağıdaki ile ise tümünü aktif edebilirsiniz.
```bash
swapon -a
```

# (LVM) Logical Volume Management

## Physical Volumes – PV Oluşturma

- oluşturmak için;


```bash
pvcreate /dev/sdb1
```
- Görmek için;


```bash
pvdisplay 
```

## Volume Groups – VG Oluşturma

- oluşturmak için;


```bash
vgcreate myvg /dev/sdb1 /dev/sdb2
```
- Görmek için;


```bash
vgdisplay 
```

# Logical Volumes – LV Oluşturma


- oluşturmak için ``300 MB boyutunda``;


```bash
lvcreate -L 300M -n mylvm myvg
```
- Görmek için;


```bash
lvdisplay 
```
### Şimdi Mount edip kullanılabilir hale getirelim
```bash
 sudo mkfs.ext4 /dev/myvg/mylvm 
```
```bash
 sudo mkdir /mylvm

```
```bash
 sudo mount /dev/myvg/mylvm /mylvm 
```

#### Bilgisayar her açıldığında kendiliğinden mount etmesini istiyorsak;
``
/etc/fstab ın içine
``

```bash
/dev/myvg/mylvm /mylvm  ext4 defaults 0 0
```