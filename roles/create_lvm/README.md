# Create LVM Role

Rola Ansible do automatycznego tworzenia grup woluminów LVM (Volume Groups) i woluminów logicznych (Logical Volumes) z automatycznym montowaniem w systemach Red Hat/CentOS/RHEL.

## Funkcjonalności

- Tworzenie grup woluminów LVM (domyślnie `vg1`)
- Tworzenie woluminów logicznych (domyślnie `lv01`)
- Automatyczne formatowanie z wybranym systemem plików
- Dodawanie wpisów do `/etc/fstab`
- Automatyczne montowanie
- Kopie zapasowe `/etc/fstab`
- Walidacja parametrów wejściowych
- Sprawdzanie dostępności dysków

## Wymagania

- Ansible 2.9+
- System Red Hat/CentOS/RHEL/Fedora
- Pakiet `lvm2` (instalowany automatycznie)
- Uprawnienia sudo (become: yes)
- Dostępny dysk do użycia w LVM

## Parametry roli

### Wymagane parametry

| Parametr | Opis |
|----------|------|
| `disk` | Nazwa dysku bez `/dev/` (np. `sdb`, `nvme0n1`) |
| `mount_destination` | Ścieżka do katalogu montowania (np. `/data`, `/backup`) |

### Opcjonalne parametry

| Parametr | Wartość domyślna | Opis |
|----------|------------------|------|
| `volume_group_name` | `vg1` | Nazwa grupy woluminów |
| `logical_volume_name` | `lv01` | Nazwa woluminu logicznego |
| `filesystem_type` | `ext4` | Typ systemu plików |
| `force_create` | `false` | Wymusza utworzenie nawet jeśli dysk jest używany |
| `backup_fstab` | `true` | Tworzy kopię zapasową `/etc/fstab` |

## Tagi

Wszystkie taski są opatrzone tagiem `create_lvm`, co pozwala na:

```bash
# Uruchomienie tylko części związanej z LVM
ansible-playbook -i inventory playbook.yml --tags create_lvm

# Pominięcie tworzenia LVM
ansible-playbook -i inventory playbook.yml --skip-tags create_lvm
```

## Przykłady użycia

### Podstawowe użycie

```yaml
---
- name: Create LVM volume
  hosts: all
  become: yes
  
  roles:
    - role: create_lvm
      disk: "sdb"
      mount_destination: "/data"
```

### Z konfiguracją parametrów

```yaml
---
- name: Create custom LVM setup
  hosts: all
  become: yes
  
  roles:
    - role: create_lvm
      disk: "nvme0n1"
      mount_destination: "/backup"
      volume_group_name: "backup_vg"
      logical_volume_name: "backup_lv"
      filesystem_type: "xfs"
      force_create: false
```

### Dla wielu dysków

```yaml
---
- name: Create multiple LVM volumes
  hosts: all
  become: yes
  
  tasks:
    - include_role:
        name: create_lvm
      vars:
        disk: "{{ item.disk }}"
        mount_destination: "{{ item.mount }}"
        volume_group_name: "{{ item.vg_name }}"
        logical_volume_name: "{{ item.lv_name }}"
      loop:
        - { disk: "sdb", mount: "/data", vg_name: "data_vg", lv_name: "data_lv" }
        - { disk: "sdc", mount: "/backup", vg_name: "backup_vg", lv_name: "backup_lv" }
```

## Uruchomienie z tagami

```bash
# Uruchomienie całej roli
ansible-playbook -i inventory lvm_playbook.yml

# Uruchomienie tylko z tagiem create_lvm
ansible-playbook -i inventory lvm_playbook.yml --tags create_lvm

# Uruchomienie bez tagów create_lvm
ansible-playbook -i inventory lvm_playbook.yml --skip-tags create_lvm
```

## Struktura utworzona przez rolę

Po wykonaniu roli zostanie utworzona następująca struktura:

```
/dev/sdb                    # Dysk fizyczny
├── /dev/vg1/lv01          # Logical Volume
└── /mount_destination     # Punkt montowania

/etc/fstab                 # Wpis dla automatycznego montowania
/etc/fstab.backup.XXXXX    # Kopia zapasowa fstab
```

## Wyjście i fakty

Rola tworzy fakt `lvm_creation_summary` z informacjami o utworzonym LVM:

```yaml
lvm_creation_summary:
  hostname: "server01"
  disk_used: "/dev/sdb"
  volume_group: "vg1"
  logical_volume: "lv01"
  device_path: "/dev/vg1/lv01"
  filesystem_type: "ext4"
  mount_point: "/data"
  available_space: "100G"
  creation_date: "2025-07-09T12:00:00Z"
```

## Bezpieczeństwo

⚠️ **OSTRZEŻENIA:**
- Rola formatuje podany dysk, co **usuwa wszystkie dane**
- Zawsze sprawdź czy dysk nie zawiera ważnych danych
- Używaj `force_create: true` tylko jeśli jesteś pewien
- Kopie zapasowe `/etc/fstab` są tworzone automatycznie

## Rozwiązywanie problemów

### Błąd: "Disk does not exist"
```bash
# Sprawdź dostępne dyski
lsblk
fdisk -l
```

### Błąd: "Disk is already used in LVM"
```bash
# Sprawdź użycie dysku
pvdisplay /dev/sdb
# Lub użyj force_create: true w playbooku
```

### Sprawdzenie statusu LVM
```bash
# Lista grup woluminów
vgdisplay

# Lista woluminów logicznych
lvdisplay

# Status montowania
df -h
```

## Licencja

MIT
