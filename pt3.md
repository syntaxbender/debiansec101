fromgpt
## 21. Yazılım İmza Doğrulama
Ubuntu, APT paket yöneticisiyle yazılım yüklerken GPG (GNU Privacy Guard) imzalarını kullanarak yazılımların güvenilir kaynaklardan geldiğini doğrular. Ancak, manuel yazılım yüklemeleri için de imza doğrulaması yapmak önemlidir.

### Manuel Olarak Yüklenen Paketler İçin İmza Doğrulama:
  - Paketleri indirirken GPG anahtarını doğrula:
    - Bir yazılım yüklemeden önce imza dosyasını indirin ve GPG anahtarıyla doğrulama yapın:
    ```
    gpg --verify <package>.sig <package>
    ```
Bu adım, yüklediğin yazılımların orijinalliğini ve bütünlüğünü doğrulamak için kullanılır.

## 22. SSH İki Adımlı Kimlik Doğrulama (MFA)
SSH için şifreleme ve anahtar tabanlı doğrulamanın yanı sıra Multi-Factor Authentication (MFA) kullanmak da önemlidir. SSH oturumlarını, sadece SSH anahtarına ek olarak bir doğrulama kodu gerektirerek güçlendirebilirsin. Örneğin, Google Authenticator kullanabilirsin (daha önce ele alınmıştı).

## 23. Dosya İzinlerini Gözden Geçirme
Linux dosya sisteminde izinler, kullanıcıların belirli dosya ve dizinlere erişimini sınırlayan önemli bir güvenlik katmanıdır. Kritik dosya ve dizinlerin izinlerini düzenli olarak gözden geçirerek yetkisiz erişimleri engelleyebilirsin.

### Kritik Dosyalar İçin İzinlerin Kontrolü:
- /etc/shadow ve /etc/passwd dosyaları için izinleri kontrol et:
  
  ```
  ls -l /etc/shadow /etc/passwd
  ```
  - **/etc/shadow**: Sadece root tarafından okunabilir olmalı.
  - **/etc/passwd**: Okuma izni olabilir ancak yazma izni sadece root'a ait olmalı.
- İzinleri gerektiği gibi düzelt:
  
  ```
  sudo chmod 640 /etc/shadow
  sudo chmod 644 /etc/passwd
  ```
- Kritik dizinlerde doğru izinleri ayarla (örneğin, /var/www için): Web sunucusu kök dizininde dosyalar kullanıcıların doğru izinlerle erişimine sunulmalıdır:
  ```
  sudo chown -R www-data:www-data /var/www
  sudo chmod -R 755 /var/www
  ```
## 24. Tam Disk Şifreleme (Full Disk Encryption)
Sunucular veya hassas veri içeren sistemler için, disk şifreleme, verilerin ele geçirilmesini önlemenin etkili bir yoludur. Ubuntu'da, disk şifrelemesi için LUKS kullanabilirsin.

### LUKS ile Disk Şifreleme:
- Disk bölümlerini şifrelemek için LUKS kullan:
  - Bir LUKS şifreli disk oluştur:
    ```
    sudo cryptsetup luksFormat /dev/sdX
    ```
  - Disk'i bağlamak için:
    ```
    sudo cryptsetup luksOpen /dev/sdX my_encrypted_disk
    ```
  -Şifreleme için otomatik bağlama işlemleri yaparak sistem başlatıldığında disk şifresini girmek zorunda kalmadan çalışabilirsin.

## 25. Network Security Hardening (Ağ Güvenliği Sıkılaştırması)
Ağ güvenliği, herhangi bir sunucu güvenliği stratejisinin temel bileşenidir. Güvenlik duvarlarının yapılandırılmasının yanı sıra ağ güvenliğini artırmak için başka adımlar da atabilirsin:

### Ağ İyileştirme Adımları:
  - IP Spoofing’i Engelle:
    - **/etc/host.conf** dosyasına şu satırı ekleyerek IP spoofing saldırılarına karşı koruma sağlayabilirsin:
      ```
      order bind,hosts
      nospoof on
      ```
  - Syn Flood Koruması:
    - TCP SYN saldırılarını önlemek için kernel parametrelerini düzenle:
      ```
      sudo sysctl -w net.ipv4.tcp_syncookies=1
      ```
  - IPv6’yı kapat: Eğer IPv6’yı kullanmıyorsan, güvenlik tehditlerini azaltmak için devre dışı bırakabilirsin:
    ```
    sudo nano /etc/sysctl.conf
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    ```



  - Ping Flood ve Smurf saldırılarını önle: Bu saldırılardan korunmak için ICMP isteklerine yanıtları sınırlandır:

    ```
    sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
    ```
  - Port Taramalarına Karşı Savunma:
    - PSAD (Port Scan Attack Detector) ile port taramalarına karşı savunma yapabilirsin:
      ```
      sudo apt install psad
      sudo psad --fw-analyze
      ```
    PSAD, gelen bağlantıları tarar ve şüpheli etkinliklerde uyarılar gönderir.

## 26. WAF (Web Application Firewall) Kullanımı
Eğer bir web sunucusu çalıştırıyorsan, Web Application Firewall (WAF) ile uygulama katmanı saldırılarına karşı koruma sağlayabilirsin. ModSecurity gibi açık kaynak WAF'lar kullanarak web trafiğini analiz edebilir ve zararlı istekleri filtreleyebilirsin.

### ModSecurity Kurulumu:
  - ModSecurity’yi yükle:

    ```
    sudo apt install libapache2-mod-security2
    ```
  - ModSecurity’yi etkinleştir:
    ```
    sudo a2enmod security2
    sudo systemctl restart apache2
    ```
  - Kuralları yapılandır ve güncelle: ModSecurity, yaygın saldırı tiplerini otomatik olarak engelleyebilir ve kendi kural setlerini de ekleyebilirsin.

## 27. DDoS Koruması
Dağıtık Hizmet Engelleme (DDoS) saldırıları, sunucuların geçici olarak kullanılamaz hale getirilmesine neden olabilir. DDoS saldırılarını önlemek için bazı adımlar atabilirsin:

### DDoS Koruma Yöntemleri:
  - Rate Limiting Yapılandır: Nginx veya Apache sunucusu üzerinde istek oranlarını sınırlayarak saldırılara karşı koruma sağlayabilirsin:
    - Nginx üzerinde rate limiting:
      ```
      http {
          limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
          ...
      }
      
      server {
          location / {
              limit_req zone=mylimit burst=10;
          }
      }
      ```
  - Fail2Ban ile HTTP Flood Koruması: Web sunucularını Fail2Ban ile entegre ederek HTTP flood saldırılarına karşı koruma sağlanabilir. Fail2Ban, belirli sayıda başarısız HTTP isteği yaptıktan sonra IP adreslerini engeller.
  - 
## 28. İzleme ve Uyarı Sistemleri
Sunucunu sürekli izlemek ve anormal durumlar karşısında uyarı almak için çeşitli izleme araçları ve çözümleri kullanabilirsin. Bu araçlar, saldırılara veya güvenlik ihlallerine karşı anlık müdahale etmene olanak sağlar.

### İzleme Araçları:
  - Nagios veya Zabbix gibi izleme araçları kullanarak sunucunun sağlığını ve güvenliğini izleyebilirsin.

  - OSSEC gibi HIDS (Host-based Intrusion Detection System) kullanarak izinsiz girişleri tespit edebilir ve uyarı alabilirsin:
    ```
    sudo apt install ossec-hids
    ```
  - Snort veya Suricata gibi IDS/IPS (Intrusion Detection/Prevention System) kullanarak ağ trafiğini analiz edip saldırıları gerçek zamanlı olarak tespit edebilirsin.

## 29. Güvenlik Güncellemelerini Zorunlu Hale Getirme
Sistem ve yazılım güncellemelerini sürekli uygulamak önemlidir. Otomatik güvenlik güncellemelerinin yanı sıra kritik güncellemeleri hızlandırmak için sistemin yapılandırmasını gözden geçirebilirsin.

- unattended-upgrades'in otomatik olarak güvenlik güncellemelerini uyguladığından emin ol.
  ```
  sudo dpkg-reconfigure unattended-upgrades
  ```
## 30. Veri Yedekleme ve Geri Yükleme Planı
Herhangi bir güvenlik olayında veri kaybını önlemek için düzenli olarak yed.... hit the limit f*ckgpt
