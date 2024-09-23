fromgpt
## 1. Firewall (UFW) Aktifleştirme ve Yapılandırma
Ubuntu’da Uncomplicated Firewall (UFW) basit bir arayüz sağlar. UFW'yi etkinleştirerek dış dünyaya açık portları kısıtlayabilirsin.

- UFW'yi etkinleştir:
```
sudo ufw enable
```
- Varsayılan olarak gelen trafiği engelle, giden trafiğe izin ver:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
- Sadece gerekli servisler için portları aç (örneğin, SSH):
```
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```
## 2. SSH Güvenliği
SSH, sunuculara erişimde en sık kullanılan yöntemlerden biridir. SSH güvenliğini artırmak için:

- **Varsayılan SSH Portunu Değiştir**: SSH'yi varsayılan port 22 yerine farklı bir porttan çalıştır:
  - **sudo nano /etc/ssh/sshd_config** dosyasını aç ve **Port 22** satırını değiştir.
  - Örneğin, **Port 2200** olarak ayarla.
- **Root Erişimini Engelle**: Root kullanıcı ile doğrudan SSH bağlantılarını kapat:
```
PermitRootLogin no
```
- **Parolasız Giriş (SSH Anahtarı)**: SSH anahtar çiftleri kullanarak şifre tabanlı girişleri kapat.
```
PasswordAuthentication no
```
- **Fail2Ban Kullanarak Brute Force Saldırılarına Karşı Koruma**:
  - Fail2Ban, belirli sayıda başarısız giriş denemesinden sonra IP adreslerini engeller:
```
sudo apt install fail2ban
```
## 3. Sudo Erişimini Kısıtla
Sudo erişimi olan kullanıcılar, sistem üzerinde tam yetkiye sahiptir. Yalnızca gerekli kullanıcıların **sudo** yetkisine sahip olduğundan emin ol.
- **Sudo kullanıcılarını kontrol etmek için**:

```
sudo cat /etc/sudoers
sudo cat /etc/group | grep sudo
```
- **Sudo işlemlerini logla**: Sudo ile yapılan tüm işlemlerin loglarını tutarak izleme yapabilirsin. **/var/log/auth.log** dosyasını inceleyebilirsin.

## 4. Güçlü Parola Politikaları
Parola politikalarını sıkılaştırmak, yetkisiz erişimleri engellemek için etkili bir yoldur.

- **Pam (Pluggable Authentication Modules)** kullanarak parola politikasını güncelle:

  - **sudo nano /etc/pam.d/common-password** dosyasında şu ayarları yap:
```
password requisite pam_pwquality.so retry=3 minlen=12 difok=3
```
Güçlü parola gereksinimleri belirle (örneğin, en az 12 karakter uzunluk, farklı karakter sınıfları).

## 5. Sudo Zaman Aşımı (Timeout)
Kullanıcıların sürekli olarak **sudo** şifresini girmelerini gerektirmek yerine, belirli bir süre boyunca geçerli kılmak için zaman aşımı süresini ayarlayabilirsin.

- **Sudo zaman aşımını ayarlamak için**:
```
sudo visudo
Defaults timestamp_timeout=5
```
  (Bu, sudo şifresi girildikten sonra 5 dakika boyunca geçerli olur.)
## 6. Sistem Güncellemeleri İçin APT Pinning
APT pinning ile belirli yazılım paketlerinin versiyonlarını sabitleyebilirsin. Örneğin, sadece güvenlik güncellemelerini alıp, kararsız veya gereksiz güncellemeleri engelleyebilirsin.

- /etc/apt/preferences.d/ altında belirli paketlere yönelik bir pin dosyası oluşturabilirsin.
## 7. AppArmor veya SELinux Kullanın
Ubuntu, AppArmor ile gelir ve uygulamaların izinlerini sıkılaştırmak için kullanılır. AppArmor ve SELinux, sistemde çalışan uygulamaların yetkilerini sınırlandırarak potansiyel güvenlik açıklarını azaltır.

- AppArmor profillerini kontrol et:

```
sudo aa-status
```
- Yeni profiller yüklemek için:
```
sudo apt install apparmor-profiles apparmor-utils
```
- SELinux kullanmak istersen, Ubuntu'da SELinux’u yükleyebilirsin:
```
sudo apt install selinux-utils selinux-basics selinux-policy-default
sudo selinux-activate
```
## 8. İzleme ve Günlükleme (Monitoring and Logging)
Sistemin güvenliğini izlemek için günlükleri takip etmek ve bir izleme aracı kurmak önemlidir.

- Syslog kullanarak sistem olaylarını izleyebilirsin:
  - Günlük dosyalar genellikle /var/log dizininde saklanır. Örneğin, auth.log SSH ve sudo etkinliklerini takip eder.
- Logwatch ile günlük raporları alabilirsin:

```
sudo apt install logwatch
```
- Auditd kullanarak güvenlik denetimlerini izleyebilirsin:

```
sudo apt install auditd audispd-plugins
sudo systemctl enable auditd
sudo systemctl start auditd
```
## 9. Güvenlikle İlgili Kernel Parametreleri
Kernel seviyesinde güvenliği artırmak için bazı parametreleri sysctl.conf dosyasında değiştirebilirsin:

- /etc/sysctl.conf dosyasına şu ayarları ekleyebilirsin:
```
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.rp_filter = 1
```
Bu ayarlar, ağ tabanlı saldırılara karşı koruma sağlar (örneğin, IP yönlendirme ve TCP SYN saldırıları).

## 10. USB Cihazlarını Kısıtla(local)
Sisteme yetkisiz USB cihazlarının takılmasını önlemek için usbguard kullanabilirsin:

- usbguard'ı yükleyip yapılandır:
```
sudo apt install usbguard
sudo systemctl start usbguard
sudo systemctl enable usbguard
```
## 11. Dosya Sistemi ve Disk Şifreleme
Sunucuda hassas veri saklanıyorsa, LUKS gibi araçlar ile disk şifreleme yapabilirsin.

- Home, swap ve diğer önemli bölümleri şifrelemek için sistem kurulumunda LUKS şifreleme seçeneğini etkinleştirebilirsin.
