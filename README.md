# Ansible Playbook Koleksiyonu

Bu repo, Ubuntu/Debian tabanlı sunucularda web sunucusu, veritabanı, Node.js uygulaması ve disk yönetimi işlemlerini otomatize eden Ansible playbook'larını içerir.

---

## Proje Yapısı

```
ansible/
├── ansible.cfg                      # Merkezi Ansible yapılandırması
├── inventory/
│   ├── hosts                        # Sunucu envanteri
│   └── group_vars/
│       ├── all.yml                  # Tüm sunucular için değişkenler
│       ├── webservers.yml           # Web sunucusu değişkenleri
│       ├── dbservers.yml            # Veritabanı değişkenleri
│       ├── appservers.yml           # Uygulama sunucusu değişkenleri
│       └── storageservers.yml       # Disk/storage değişkenleri
├── vault/
│   └── secrets.yml                  # ŞİFRELENMESİ GEREKEN hassas veriler
├── roles/
│   ├── common/                      # Tüm sunucularda çalışan temel görevler
│   ├── apache/                      # Apache web sunucusu kurulumu
│   ├── nodejs/                      # Node.js + PM2 uygulama sunucusu
│   ├── mysql/                       # MySQL kurulum ve yapılandırma
│   ├── mysql_relocate/              # MySQL veri dizinini yeniden konumlandırma
│   ├── mysql_restore/               # MySQL dump'tan veritabanı geri yükleme
│   ├── php_mysql/                   # PHP + MySQL bağlantı sayfası
│   ├── lvm/                         # LVM kurulumu (PV→VG→LV→mount)
│   ├── lvm_extend/                  # Mevcut LVM'i yeni diskle genişletme
│   ├── disk/                        # Düz disk bölümleme ve mount
│   └── network_info/                # Ağ bilgilerini topla ve web'de sun
├── files/
│   └── dump.sql                     # MySQL örnek dump dosyası
│
│   — Playbook'lar —
├── site.yml                         # Ana playbook (tüm roller)
├── web.yml                          # Web sunucusu
├── db.yml                           # Veritabanı kurulumu
├── db_restore.yml                   # Veritabanı geri yükleme
├── db_relocate.yml                  # MySQL dizin taşıma
├── db_connect.yml                   # PHP-MySQL bağlantı sayfası
├── lvm.yml                          # LVM kurulumu
├── extend_lvm.yml                   # LVM genişletme
├── disk.yml                         # Disk bölümleme
└── network_info.yml                 # Ağ bilgileri toplama
```

---

## Hızlı Başlangıç

### 1. Inventory'yi Düzenle

`inventory/hosts` dosyasını açıp sunucu IP adreslerini güncelle:

```ini
[webservers]
192.168.1.XX

[dbservers]
192.168.1.XX
```

### 2. Vault ile Şifreleri Koru

`vault/secrets.yml` dosyasındaki şifreleri gerçek değerlerle doldur, ardından şifrele:

```bash
# Şifreleri düzenle
nano vault/secrets.yml

# Dosyayı şifrele
ansible-vault encrypt vault/secrets.yml

# Daha sonra düzenlemek için
ansible-vault edit vault/secrets.yml
```

### 3. SSH Anahtarını Oluştur

```bash
ssh-keygen -t ed25519 -f ~/.ssh/ansible -C "ansible"
ssh-copy-id -i ~/.ssh/ansible.pub bora@192.168.1.XX
```

---

## Playbook Kullanımı

### Web Sunucusu (Apache)

```bash
# Apache'yi kur
ansible-playbook web.yml

# Sadece yapılandırmayı güncelle
ansible-playbook web.yml --tags config

# Belirli bir host için
ansible-playbook web.yml --limit 192.168.1.45
```

### Veritabanı (MySQL)

```bash
# MySQL kur ve yapılandır
ansible-playbook db.yml --ask-vault-pass

# Sadece güvenlik adımları
ansible-playbook db.yml --ask-vault-pass --tags security

# Dump'tan geri yükle
ansible-playbook db_restore.yml --ask-vault-pass

# MySQL dizinlerini LVM'e taşı (önce lvm.yml çalıştır)
ansible-playbook db_relocate.yml --ask-vault-pass
```

### PHP-MySQL Bağlantı Sayfası

```bash
ansible-playbook db_connect.yml --ask-vault-pass
```

### Node.js Uygulama Sunucusu

```bash
# Node.js + PM2 kur ve uygulamayı başlat
ansible-playbook site.yml --tags nodejs --ask-vault-pass
```

### Disk ve LVM Yönetimi

```bash
# Düz disk bölümleme (LVM'siz)
ansible-playbook disk.yml

# LVM kurulumu
ansible-playbook lvm.yml

# Mevcut LVM'i yeni diskle genişlet
ansible-playbook extend_lvm.yml

# Farklı disk belirterek çalıştır
ansible-playbook lvm.yml -e "lvm_disk_device=/dev/sdc lvm_vg_name=data_vg"
```

### Ağ Bilgileri Sayfası

```bash
# Sunucunun IP, hostname ve saat bilgisini Apache ile sun
ansible-playbook network_info.yml
```

### Tüm Sistemi Bir Seferde Kur

```bash
ansible-playbook site.yml --ask-vault-pass
```

---

## Etiket (Tag) Referansı

| Etiket       | Açıklama                            |
|:-------------|:------------------------------------|
| `always`     | Her zaman çalışır (cache güncelleme)|
| `apache`     | Apache görevleri                    |
| `mysql`      | MySQL görevleri                     |
| `nodejs`     | Node.js görevleri                   |
| `pm2`        | PM2 servis yönetimi                 |
| `lvm`        | LVM görevleri                       |
| `disk`       | Disk bölümleme görevleri            |
| `network`    | Ağ bilgisi görevleri                |
| `packages`   | Sadece paket kurulumu               |
| `config`     | Sadece yapılandırma değişiklikleri  |
| `security`   | Güvenlik ve kullanıcı yönetimi      |
| `restore`    | Veritabanı geri yükleme             |
| `relocate`   | Dizin taşıma işlemleri              |

---

## Değişken Önceliği

Değişkenler aşağıdaki öncelik sıralamasıyla yüklenir (yukarı → daha yüksek öncelik):

```
roles/<rol>/defaults/main.yml   ← en düşük
inventory/group_vars/all.yml
inventory/group_vars/<group>.yml
-e "key=value" (komut satırı)   ← en yüksek
```

Sunucuya özel değerleri `inventory/host_vars/<ip>.yml` dosyasına ekleyebilirsin.

---

## Gereksinimler

| Yazılım        | Sürüm   |
|:---------------|:--------|
| Ansible        | ≥ 2.14  |
| Python         | ≥ 3.8   |
| Hedef OS       | Ubuntu 20.04+ / Debian 11+ |

### Gerekli Ansible Koleksiyonları

```bash
ansible-galaxy collection install community.mysql
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

---

## Güvenlik Notları

- `vault/secrets.yml` dosyasını **asla şifrelemeden** commit etme.
- `.gitignore` dosyasına `vault/secrets.yml` veya `.vault_pass` eklemeyi düşün.
- Vault şifresini CI/CD sistemlerinde ortam değişkeni olarak sakla: `ANSIBLE_VAULT_PASSWORD_FILE`.

---

## Notlar

- `mysql2.yml` (eski): RHEL/CentOS için hazırlanmıştı ve şirkete özel bir pip mirror kullanıyordu. Bu işlevsellik için gerekirse `roles/mysql/tasks/` altında ayrı bir `rhel.yml` görevi eklenebilir.
- Tüm servis yeniden başlatmaları handler üzerinden yapılır; idempotent çalışmayı garanti eder.
- `gather_facts: false` olan playbook'larda (network_info) sadece gerekli bilgiler `ansible.builtin.command` ile toplanır.
