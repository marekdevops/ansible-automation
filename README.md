# Ansible Automation - Disk Management Roles

Zestaw ról Ansible do zarządzania dyskami w systemach Linux, szczególnie Red Hat/CentOS/RHEL. Repozytorium zawiera role do analizy użycia dysków oraz automatycznego tworzenia struktur LVM.

## 📋 Zawartość repozytorium

### 🔍 Rola: `disk_usage_report`
Zaawansowana analiza i raportowanie użycia dysków w systemie.

**Funkcjonalności:**
- Analiza wszystkich urządzeń blokowych (`lsblk`)
- Wykrywanie nieużywanych dysków
- Raportowanie zamontowanych systemów plików
- Generowanie strukturalnych faktów podsumowujących

### ⚙️ Rola: `create_lvm`
Automatyczne tworzenie struktur LVM z montowaniem.

**Funkcjonalności:**
- Tworzenie Physical Volume, Volume Group i Logical Volume
- Formatowanie z wybranym systemem plików
- **Automatyczne tworzenie katalogów montowania** (w tym zagnieżdżonych)
- Automatyczne montowanie i wpisy w `/etc/fstab`
- Walidacja parametrów i bezpieczeństwo
- Sprawdzanie czy katalog nie jest już zamontowany
- Inteligentne obsługiwanie istniejących struktur LVM

## 🎯 Przypadki użycia

### 1. **Analiza dysków w systemie**

```bash
# Podstawowa analiza wszystkich dysków
ansible-playbook -i inventory playbooks/disk_usage_report_playbook.yml

# Tylko analiza lsblk z tagiem
ansible-playbook -i inventory playbooks/disk_usage_report_playbook.yml --tags lsblk

# Pomiń analizę dysków
ansible-playbook -i inventory some_playbook.yml --skip-tags lsblk
```

### 2. **Tworzenie LVM na nowym dysku**

```bash
# Podstawowe utworzenie LVM
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml

# Tylko operacje LVM z tagiem
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --tags create_lvm

# Pomiń tworzenie LVM
ansible-playbook -i inventory some_playbook.yml --skip-tags create_lvm
```

### 3. **Kompletny workflow analizy i tworzenia LVM**

```bash
# Pełny workflow - analiza + LVM
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml

# Tylko analiza dysków
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml --tags disk_analysis

# Tylko operacje lsblk
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml --tags lsblk

# Tylko tworzenie LVM (jeśli odkomentowane)
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml --tags create_lvm
```

### 4. **Auditowanie serwerów - tylko raportowanie**

```bash
# Sprawdzenie dysków bez zmian w systemie
ansible-playbook -i inventory playbooks/disk_usage_report_playbook.yml --tags lsblk --check

# Analiza dla wielu serwerów
ansible-playbook -i production_inventory playbooks/disk_usage_report_playbook.yml --tags lsblk
```

### 5. **Przygotowanie serwerów z wieloma dyskami**

```bash
# Wykorzystanie playbooka dla wielu LVM
ansible-playbook -i inventory playbooks/create_multiple_lvm.yml --tags create_lvm
```

### 6. **Archiwizacja i backup katalogów**

```bash
# Podstawowa archiwizacja katalogu
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/path/to/directory"

# Archiwizacja z niestandardową nazwą
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/etc archive_name_override=system_config_backup.tar.gz"

# Archiwizacja z automatycznym usuwaniem pliku zdalnego
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/var/log cleanup=true"

# Archiwizacja z wysoką kompresją
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/home/user/documents compression=9"
```

## 📁 Struktura plików

```
├── roles/
│   ├── disk_usage_report/          # Rola analizy dysków
│   │   ├── tasks/main.yml         # Taski z tagiem 'lsblk'
│   │   ├── defaults/main.yml      # Zmienne domyślne
│   │   ├── vars/main.yml          # Zmienne robocze
│   │   ├── meta/main.yml          # Metadane roli
│   │   └── README.md              # Dokumentacja roli
│   └── create_lvm/                # Rola tworzenia LVM
│       ├── tasks/main.yml         # Taski z tagiem 'create_lvm'
│       ├── defaults/main.yml      # Zmienne domyślne
│       ├── vars/main.yml          # Zmienne robocze
│       ├── handlers/main.yml      # Handlery
│       ├── meta/main.yml          # Metadane roli
│       └── README.md              # Dokumentacja roli
├── playbooks/
│   ├── disk_usage_report_playbook.yml     # Podstawowy playbook analizy
│   ├── disk_usage_report_custom.yml       # Analiza z konfiguracją
│   ├── create_lvm_playbook.yml            # Podstawowy playbook LVM
│   ├── create_lvm_custom.yml              # LVM z konfiguracją
│   ├── create_multiple_lvm.yml            # Tworzenie wielu LVM
│   ├── complete_disk_workflow.yml         # Kompletny workflow
│   ├── create_virtual_disk.yml            # Tworzenie wirtualnego dysku
│   ├── diagnose_disk_issue.yml            # Diagnostyka problemów
│   ├── demo_directory_creation.yml        # Demo możliwości tworzenia katalogów
│   ├── archive_directory.yml              # Archiwizacja katalogów z kompresją
│   └── archive_directory_demo.yml         # Demo archiwizacji
├── inventory                               # Plik inventory
└── README.md                              # Ten plik
```

## 🏷️ System tagów

### **Tag: `lsblk`**
Uruchamia wszystkie operacje związane z analizą dysków.

```bash
# Przykłady użycia
ansible-playbook playbook.yml --tags lsblk
ansible-playbook playbook.yml --skip-tags lsblk
```

**Zawiera:**
- Pobieranie informacji o urządzeniach blokowych
- Parsowanie danych JSON z lsblk
- Wykrywanie nieużywanych dysków
- Generowanie raportów

### **Tag: `create_lvm`**
Uruchamia wszystkie operacje związane z tworzeniem LVM.

```bash
# Przykłady użycia
ansible-playbook playbook.yml --tags create_lvm
ansible-playbook playbook.yml --skip-tags create_lvm
```

**Zawiera:**
- Walidację parametrów
- Tworzenie PV, VG, LV
- Formatowanie systemów plików
- Montowanie i wpisy w fstab

### **Tag: `disk_analysis`**
Kompleksowa analiza dysków (używany w workflow).

```bash
ansible-playbook complete_disk_workflow.yml --tags disk_analysis
```

### **Tag: `archive`**
Operacje archiwizacji i kompresji katalogów.

```bash
# Przykłady użycia
ansible-playbook playbook.yml --tags archive
ansible-playbook playbook.yml --skip-tags archive
```

**Zawiera:**
- Walidację przestrzeni dyskowej
- Tworzenie archiwów tar.gz
- Kompresję z konfigurowalnymi poziomami
- Kopiowanie na localhost
- Opcjonalne czyszczenie plików zdalnych

## 📖 Szczegółowe przykłady

### **Przykład 1: Audyt dysków w środowisku produkcyjnym**

```yaml
---
- name: Production disk audit
  hosts: production_servers
  gather_facts: yes
  become: yes
  
  roles:
    - disk_usage_report
```

**Uruchomienie:**
```bash
ansible-playbook -i production_inventory audit_playbook.yml --tags lsblk
```

### **Przykład 2: Przygotowanie nowego serwera z dyskiem danych**

```yaml
---
- name: Setup new server with data disk
  hosts: new_servers
  gather_facts: yes
  become: yes
  
  tasks:
    # Krok 1: Analiza dostępnych dysków
    - include_role:
        name: disk_usage_report
      tags: 
        - setup
        - lsblk
        
    # Krok 2: Utworzenie LVM na określonym dysku
    - include_role:
        name: create_lvm
      vars:
        disk: "sdb"
        mount_destination: "/data"
        volume_group_name: "data_vg"
        logical_volume_name: "data_lv"
        filesystem_type: "ext4"
      tags:
        - setup
        - create_lvm
      when: unused_disks is defined and unused_disks | length > 0
```

**Uruchomienie:**
```bash
# Tylko analiza
ansible-playbook -i inventory setup_server.yml --tags lsblk

# Pełne przygotowanie
ansible-playbook -i inventory setup_server.yml --tags setup

# Tylko LVM
ansible-playbook -i inventory setup_server.yml --tags create_lvm
```

### **Przykład 3: Serwer backupowy z wieloma dyskami**

```yaml
---
- name: Setup backup server with multiple disks
  hosts: backup_servers
  gather_facts: yes
  become: yes
  
  tasks:
    - include_role:
        name: disk_usage_report
      tags: [backup_setup, lsblk]
        
    - include_role:
        name: create_lvm
      vars:
        disk: "{{ item.disk }}"
        mount_destination: "{{ item.mount }}"
        volume_group_name: "{{ item.vg_name }}"
        logical_volume_name: "{{ item.lv_name }}"
        filesystem_type: "{{ item.fs_type | default('ext4') }}"
      loop:
        - { disk: "sdb", mount: "/backup/daily", vg_name: "daily_vg", lv_name: "daily_lv" }
        - { disk: "sdc", mount: "/backup/weekly", vg_name: "weekly_vg", lv_name: "weekly_lv", fs_type: "xfs" }
        - { disk: "sdd", mount: "/backup/monthly", vg_name: "monthly_vg", lv_name: "monthly_lv" }
      tags: [backup_setup, create_lvm]
```

**Uruchomienie:**
```bash
# Analiza dysków przed konfiguracją
ansible-playbook -i inventory backup_setup.yml --tags lsblk

# Konfiguracja tylko dysków backup
ansible-playbook -i inventory backup_setup.yml --tags backup_setup

# Tylko tworzenie LVM
ansible-playbook -i inventory backup_setup.yml --tags create_lvm
```

### **Przykład 4: Backup konfiguracji systemu**

```yaml
---
- name: System configuration backup
  hosts: production_servers
  gather_facts: yes
  become: yes
  
  tasks:
    # Backup konfiguracji systemu
    - include: playbooks/archive_directory.yml
      vars:
        source_dir: "/etc"
        archive_name_override: "{{ inventory_hostname }}_system_config_{{ ansible_date_time.date }}.tar.gz"
        local_dest: "./backups/system_configs"
        compression: "9"
        cleanup: true
      tags: [backup, system_config]
    
    # Backup logów
    - include: playbooks/archive_directory.yml
      vars:
        source_dir: "/var/log"
        archive_name_override: "{{ inventory_hostname }}_logs_{{ ansible_date_time.date }}.tar.gz"
        local_dest: "./backups/logs"
        compression: "6"
        cleanup: true
      tags: [backup, logs]
```

**Uruchomienie:**
```bash
# Backup tylko konfiguracji
ansible-playbook -i inventory system_backup.yml --tags system_config

# Backup tylko logów
ansible-playbook -i inventory system_backup.yml --tags logs

# Pełny backup
ansible-playbook -i inventory system_backup.yml --tags backup
```

## ⚡ Szybkie polecenia

### **Często używane kombinacje tagów:**

```bash
# 1. Sprawdź wszystkie dyski w klastrze
ansible-playbook -i cluster_inventory playbooks/disk_usage_report_playbook.yml --tags lsblk

# 2. Przygotuj nowe serwery (analiza + LVM)
ansible-playbook -i new_servers playbooks/complete_disk_workflow.yml

# 3. Tylko LVM bez analizy (dla znanych konfiguracji)
ansible-playbook -i inventory playbooks/create_lvm_custom.yml --tags create_lvm

# 4. Utwórz LVM z zagnieżdżonymi katalogami
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --extra-vars "disk=loop6 mount_destination=/opt/applications/tomcat"

# 5. Audyt bez zmian (dry-run)
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml --tags lsblk --check

# 6. Pomiń operacje na dyskach (dla innych zadań)
ansible-playbook -i inventory full_server_setup.yml --skip-tags lsblk,create_lvm

# 7. Archiwizuj konfigurację systemu
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/etc cleanup=true"

# 8. Backup z wysoką kompresją
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/var/log compression=9 cleanup=true"
```

## 🔧 Konfiguracja środowiska

### **Wymagania:**
- Ansible 2.9+
- Python 3.6+
- Systemy docelowe: RHEL/CentOS/Fedora/Ubuntu
- Pakiety: `lvm2` (instalowany automatycznie)

### **Instalacja:**
```bash
# Klonowanie repozytorium
git clone <repository-url> ansible-automation
cd ansible-automation

# Przygotowanie inventory
cp inventory.example inventory
# Edytuj inventory według potrzeb

# Konfiguracja jest gotowa (ansible.cfg już skonfigurowany)

# Test połączenia
ansible all -m ping

# Test z sudo (jeśli potrzebne)
ansible-playbook playbooks/disk_usage_report_playbook.yml --tags lsblk --ask-become-pass
```

### **Przykładowy inventory:**
```ini
[production]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11

[staging]
staging1 ansible_host=192.168.1.20

[new_servers]
new1 ansible_host=192.168.1.30

[all:vars]
ansible_user=ansible
ansible_become=yes
```

## 🔒 Bezpieczeństwo

⚠️ **OSTRZEŻENIA:**
- Operacje LVM **usuwają wszystkie dane** z dysków
- Zawsze wykonuj `--check` przed rzeczywistym uruchomieniem
- Sprawdzaj wyniki analizy dysków przed utworzeniem LVM
- Kopie zapasowe `/etc/fstab` są tworzone automatycznie

### **Bezpieczne praktyki:**
```bash
# 1. Zawsze najpierw analiza
ansible-playbook -i inventory playbooks/complete_disk_workflow.yml --tags lsblk

# 2. Dry-run przed LVM
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --check

# 3. Test na pojedynczym serwerze
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --limit server1

# 4. Weryfikacja po wykonaniu
ansible-playbook -i inventory playbooks/disk_usage_report_playbook.yml --tags lsblk
```

## 🔧 Rozwiązywanie problemów

### **Problem: "Disk /dev/sdb does not exist"**

Jeśli otrzymujesz błąd że dysk nie istnieje, mimo że był wykazany w raporcie:

```bash
# 1. Sprawdź rzeczywisty stan dysków
ansible-playbook -i inventory playbooks/diagnose_disk_issue.yml --tags lsblk

# 2. Sprawdź konkretny dysk
ansible-playbook -i inventory playbooks/diagnose_disk_issue.yml --extra-vars "validate_disk=sdb" --tags disk_validation

# 3. Pokaż dostępne dyski
ansible-playbook -i inventory playbooks/disk_usage_report_playbook.yml --tags lsblk
```

**Częste przyczyny:**
- Dysk był w raporcie z innego systemu/momentu
- Dysk został usunięty lub odłączony
- Błędna nazwa dysku (sprawdź `lsblk` bezpośrednio)

### **Problem: Katalog montowania nie istnieje**

```bash
# Rola automatycznie tworzy katalogi - nawet zagnieżdżone!
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --extra-vars "disk=loop6 mount_destination=/path/that/does/not/exist"

# Przykłady automatycznego tworzenia:
# /tomcat - prosty katalog
# /opt/applications/tomcat - zagnieżdżone katalogi
# /var/lib/mysql/data - wielopoziomowe ścieżki
```

**Funkcje automatycznego tworzenia katalogów:**
- Tworzy wszystkie katalogi nadrzędne (`recurse: yes`)
- Ustawia właściwe uprawnienia (755, root:root)
- Sprawdza czy katalog nie jest już zamontowany
- Bezpiecznie obsługuje istniejące katalogi

### **Problem: Brak dostępnych dysków do LVM**

```bash
# Utwórz wirtualny dysk do testów
ansible-playbook -i inventory playbooks/create_virtual_disk.yml

# Następnie użyj loop6 do testów
ansible-playbook -i inventory playbooks/create_lvm_playbook.yml --extra-vars "disk=loop6 mount_destination=/test"
```

### **Problem: "Bad size specification" w LVM**

Naprawiono poprzez użycie `100%FREE` zamiast parsowania rozmiaru z `vgdisplay`.

### **Problem: Role nie są znajdowane**

```bash
# Sprawdź czy ansible.cfg jest skonfigurowany
cat ansible.cfg

# Uruchom z pełną ścieżką
ansible-playbook -i inventory ./playbooks/nazwa_playbook.yml
```

### **Diagnostyka systemowa**

```bash
# Pełna diagnostyka dysków
ansible-playbook -i inventory playbooks/diagnose_disk_issue.yml

# Pokazanie rekomendacji
ansible-playbook -i inventory playbooks/diagnose_disk_issue.yml --tags recommendations
```

### **Problem: Błędy archiwizacji**

```bash
# Sprawdź dostępne miejsce przed archiwizacją
df -h /tmp

# Demo archiwizacji
ansible-playbook -i inventory playbooks/archive_directory_demo.yml --tags demo

# Test z walidacją miejsca
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/path" --tags validation

# Archiwizacja z debugowaniem
ansible-playbook -i inventory playbooks/archive_directory.yml --extra-vars "source_dir=/path" -vv
```

**Częste problemy z archiwizacją:**
- Brak miejsca w /tmp - playbook sprawdza automatycznie
- Brak uprawnień do katalogu źródłowego - użyj `become: yes`
- Błędy kopiowania na localhost - sprawdź uprawnienia do katalogu docelowego
- Duże pliki - użyj wyższej kompresji (`compression=9`)

---

**Autor:** Marek  
**Licencja:** MIT  
**Wersja:** 1.0
