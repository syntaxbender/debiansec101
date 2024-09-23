12. İki Faktörlü Kimlik Doğrulama (2FA)
SSH bağlantıları için iki faktörlü kimlik doğrulama (2FA) eklemek, ekstra bir güvenlik katmanı sağlar. Ubuntu'da Google Authenticator veya başka bir 2FA yöntemi kullanarak SSH oturumlarını koruyabilirsin.

Google Authenticator ile 2FA Kurulumu:
Google Authenticator PAM modülünü yükle:

bash
Copy code
sudo apt install libpam-google-authenticator
2FA için PAM dosyasını düzenle:

bash
Copy code
sudo nano /etc/pam.d/sshd
Dosyanın sonuna şu satırı ekle:

bash
Copy code
auth required pam_google_authenticator.so
SSH ayarlarını düzenle:

bash
Copy code
sudo nano /etc/ssh/sshd_config
Şu ayarları değiştir:

bash
Copy code
ChallengeResponseAuthentication yes
Google Authenticator'ı ayarla: SSH kullanıcısında şu komutu çalıştırarak 2FA kodlarını al:

bash
Copy code
google-authenticator
SSH hizmetini yeniden başlat:

bash
Copy code
sudo systemctl restart sshd
Artık, SSH oturumlarına giriş yaparken bir doğrulama kodu (OTP) girilmesi gerekecek.

13. Port Knocking
Port knocking, belirli bir sırayla belirlenen portlara erişilmesi sonucu bir güvenlik duvarı kuralının tetiklenmesi prensibine dayanır. Bu, yalnızca doğru sırayla knock yapan kişilerin belirli portlara erişim izni almasını sağlar (örneğin, SSH).

Port Knocking Kurulumu:
Knockd’yi yükle:

bash
Copy code
sudo apt install knockd
Port Knocking ayarlarını yapılandır: Knockd yapılandırma dosyasını düzenle:

bash
Copy code
sudo nano /etc/knockd.conf
Örnek ayar:

ini
Copy code
[openSSH]
sequence = 7000,8000,9000
seq_timeout = 5
command = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
tcpflags = syn

[closeSSH]
sequence = 9000,8000,7000
seq_timeout = 5
command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
tcpflags = syn
Knockd'yi başlat:

bash
Copy code
sudo systemctl start knockd
sudo systemctl enable knockd
Portları knock yapmak için:

bash
Copy code
knock <server_ip> 7000 8000 9000
Bu şekilde, yalnızca knock dizisini doğru yapan IP adresleri SSH portuna erişim sağlar.

14. Kernel Güncellemeleri İçin Livepatch Kullanımı
Kernel güvenlik açıklarını gidermek ve sistem yeniden başlatma gerektirmeden kernel güncellemelerini uygulamak için Canonical'ın Livepatch servisini kullanabilirsin.

Livepatch Ayarları:
Livepatch'i etkinleştirmek için:
Ubuntu One hesabı oluştur.
Livepatch token'ını al ve şu komutla etkinleştir:
bash
Copy code
sudo snap install canonical-livepatch
sudo canonical-livepatch enable <your-token>
Bu işlem, kritik kernel güncellemelerini sistem yeniden başlatma gerektirmeden uygular.

15. Log'ları Remote (Uzak) Sunucuya Gönderme
Saldırganların izlerini silmelerini önlemek için yerel sistem log'larını uzak bir log sunucusuna yönlendirmek iyi bir uygulamadır.

Rsyslog ile Uzak Log Sunucusuna Yönlendirme:
Rsyslog yapılandırmasını düzenle:

bash
Copy code
sudo nano /etc/rsyslog.conf
Uzak log sunucusuna log gönderimini etkinleştir: Şu satırı ekle:

bash
Copy code
*.* @<log_server_ip>:514
Rsyslog'u yeniden başlat:

bash
Copy code
sudo systemctl restart rsyslog
16. SUID ve SGID Bit'leri Kontrol Etme
Bazı dosyalar SUID (Set User ID) veya SGID (Set Group ID) bitlerine sahiptir ve bunlar güvenlik riskleri oluşturabilir. SUID ve SGID bit'leri ayarlanmış dosyaları belirleyip inceleyebilirsin.

SUID ve SGID Bit'leri Tarama:
SUID dosyalarını taramak için:

bash
Copy code
find / -perm /4000 -type f 2>/dev/null
SGID dosyalarını taramak için:

bash
Copy code
find / -perm /2000 -type f 2>/dev/null
Bu dosyaları inceleyip gerçekten gerekli olmayanların SUID/SGID bitlerini kaldırabilirsin:

bash
Copy code
sudo chmod u-s <dosya>
sudo chmod g-s <dosya>
17. Rootkit Taraması (Chkrootkit veya rkhunter)
Sistemde rootkit veya kötü niyetli yazılımları tespit etmek için chkrootkit veya rkhunter gibi araçlar kullanabilirsin.

Chkrootkit Kurulumu ve Kullanımı:
Chkrootkit'i yükle:

bash
Copy code
sudo apt install chkrootkit
Taramayı çalıştır:

bash
Copy code
sudo chkrootkit
Rkhunter Kurulumu ve Kullanımı:
Rkhunter'i yükle:

bash
Copy code
sudo apt install rkhunter
Taramayı çalıştır:

bash
Copy code
sudo rkhunter --check
18. Docker Kullananlar İçin Güvenlik Önlemleri
Eğer Docker kullanıyorsan, Docker güvenliğini artırmak için şu önlemleri alabilirsin:

Varsayılan kök kullanıcısını sınırlandır (Rootless Docker): Docker'ı kök kullanıcı yerine normal kullanıcı olarak çalıştır.
Kapsayıcıları ağdan izole et: Gereksiz ağ bağlantılarına izin verme ve izole ağlar kullan.
Seccomp, AppArmor ve SELinux profilleri kullanarak kapsayıcıların yetkilerini sınırlandır.
19. Exploit Engelleyiciler (ASLR, Stack Canaries)
Kernel tabanlı güvenlik özelliklerini etkinleştirmek saldırılara karşı ek bir katman sağlar. ASLR (Address Space Layout Randomization) ve Stack Canaries, bellekle ilgili saldırılara karşı etkili olabilir.

ASLR'nin Etkin Olduğunu Kontrol Etme:
ASLR'nin durumunu kontrol et:

bash
Copy code
cat /proc/sys/kernel/randomize_va_space
0: Devre dışı
1: Kısmen etkin
2: Tam etkin (önerilen)
ASLR'yi tam etkinleştirmek için:

bash
Copy code
sudo sysctl -w kernel.randomize_va_space=2
20. Disk Kotaları Ayarlama
Kullanıcıların veya uygulamaların diskte çok fazla yer kaplamasını engellemek için disk kotaları ayarlayabilirsin. Bu, disk taşmalarına ve dosya sisteminin yanlışlıkla dolmasına karşı koruma sağlar.

Disk Kotalarını Kurma:
Quota paketini yükle:

bash
Copy code
sudo apt install quota
Quota'yı etkinleştir ve yapılandır:

/etc/fstab dosyasındaki ilgili bölüme usrquota ve grpquota ekle.
Şu komutla kota verilerini oluştur:
bash
Copy code
sudo quotacheck -cum /mount_point
sudo quotaon /mount_point
Kullanıcılara kota ayarla:

bash
Copy code
sudo edquota -u username
Bu yöntemlerle sistemin güvenlik katmanlarını daha da artırabilirsin. Güvenlik sıkılaştırmaları, saldırı vektörlerini azaltarak potansiyel tehditlerin sistem üzerinde etkili olmasını engelleyecektir.
