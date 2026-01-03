Evet, Linux tarafında **bir uygulamanın sistemde yaptığı neredeyse tüm aktiviteleri** (dosya işlemleri, network, process davranışı, açılan portlar vs.) **tek tek ya da birlikte** izlemeni sağlayan oldukça güçlü araçlar var. Bunu genelde **katmanlı** yaparız; çünkü tek bir tool her şeyi “temiz ve eksiksiz” vermez.

Aşağıda **güvenlik araştırmacısı / malware analyst bakış açısıyla** en pratik ve güçlü kombinasyonu anlatıyorum.

---

## 1️⃣ System Call Seviyesi (En Düşük Seviye – En Net Gerçeklik)

### 🔹 `strace` – hızlı ve klasik

Uygulamanın yaptığı **tüm syscall’ları** görürsün.

```bash
strace -ff -o trace.log ./target_app
```

Özellikle şunları yakalamak için:

```bash
strace -f \
  -e trace=file,network,process \
  ./target_app
```

Gördüklerin:

* `open/openat` → hangi dosyaları okuyor/yazıyor
* `connect/sendto/recvfrom` → network
* `fork/execve` → child process
* `chmod/chown` → privilege davranışları

⚠️ Dezavantaj:

* Gürültülü
* Kernel exploit veya `ptrace` engelleme varsa körleşebilir

---

## 2️⃣ Kernel Seviyesi (Modern & Güçlü)

### 🔹 `sysdig` ⭐ (çok öneririm)

Kernel event’lerini **kural bazlı** ve **okunabilir** şekilde verir.

```bash
sysdig proc.name=target_app
```

Dosya aktiviteleri:

```bash
sysdig evt.type in (open,openat,creat,unlink)
```

Network:

```bash
sysdig evt.type=connect
```

📌 Avantaj:

* strace’e göre **çok daha temiz**
* Container & host fark etmez
* Rootkit olmayan malware için çok net

---

## 3️⃣ eBPF Tabanlı İzleme (Profesyonel Seviye)

### 🔹 `bcc` / `bpftrace`

Gerçek zamanlı kernel tracing.

Örnek: hangi dosyaları açıyor?

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'
```

Network:

```bash
bpftrace -e 'tracepoint:syscalls:sys_enter_connect { printf("%s\n", comm); }'
```

📌 Avantaj:

* **Anti-debug bypass edemez**
* Performans çok iyi
* Stealth

📌 Dezavantaj:

* Kernel & eBPF bilgisi ister

---

## 4️⃣ Dosya Sistemi Takibi

### 🔹 `auditd` (adli analiz için ideal)

```bash
auditctl -w /etc -p rwxa -k etc_watch
```

Sonra:

```bash
ausearch -k etc_watch
```

Bu sayede:

* Kim hangi dosyayı oluşturdu
* Ne zaman sildi
* Permission değişti mi

---

## 5️⃣ Network Davranışı

### 🔹 Canlı bağlantılar

```bash
ss -tunap | grep target_app
```

### 🔹 Paket seviyesi (çok kritik)

```bash
tcpdump -i any -w traffic.pcap
```

Sonra:

```bash
wireshark traffic.pcap
```

📌 Malware C2, DNS tunneling, TLS fingerprint vs. burada çıkar.

---

## 6️⃣ Process & Persistence Davranışı

### Açılan process’ler:

```bash
pstree -ap
```

### Cron, systemd, autorun kontrolü:

```bash
crontab -l
ls /etc/cron.*
systemctl list-unit-files
```

---

## 7️⃣ Sandbox Yaklaşımı (En Sağlıklısı)

Gerçek sistemi kirletmemek için:

### 🔹 `firejail`

```bash
firejail --net=none ./target_app
```

### 🔹 Container / VM

* Docker + sysdig
* QEMU/KVM snapshot

---

## 8️⃣ Hepsini Birleştiren Framework’ler

### ⭐ `falco`

* Sysdig tabanlı
* “Bu davranış şüpheli mi?” diye bağırır

### ⭐ Malware analizi için:

* **Cuckoo Sandbox (Linux agent ile)**
* **CAPEv2**

---

## 🎯 Önerilen Minimal Setup (Pratik)

Eğer “bugün analiz yapacağım” diyorsan:

```text
1) firejail / VM
2) sysdig
3) tcpdump
4) strace (gerekirse)
```

---

## İstersen şunları da yapabilirim:

* 🔍 **Belirli bir senaryo için komut seti** (örneğin credential stealing)
* 🧪 **Anti-debug / anti-analysis davranışlarını nasıl tespit edersin**
* 🧰 **Malware sample için otomatik script**

Ne analiz edeceğini (normal app mi, şüpheli binary mi, container mı) söylersen setup’ı netleştireyim.
