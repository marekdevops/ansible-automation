# Disk Usage Report Role

Rola Ansible do generowania raportów użycia dysków i wykrywania nieużywanych dysków w systemach Linux.

## Funkcjonalności

- Pobieranie informacji o wszystkich urządzeniach blokowych za pomocą `lsblk`
- Analiza zamontowanych systemów plików
- Wykrywanie nieużywanych dysków (niepartycjonowanych i niezamontowanych)
- Generowanie szczegółowych raportów
- Obsługa tagów do selektywnego wykonywania

## Wymagania

- Ansible 2.9+
- System Linux z dostępnymi narzędziami `lsblk` i `df`
- Uprawnienia sudo (become: yes)

## Zmienne roli

### Defaults (roles/disk_usage_report/defaults/main.yml)

| Zmienna | Wartość domyślna | Opis |
|---------|------------------|------|
| `show_detailed_output` | `true` | Wyświetla szczegółowe informacje o urządzeniach |
| `check_unused_disks` | `true` | Sprawdza nieużywane dyski |
| `minimum_disk_size_gb` | `1` | Minimalny rozmiar dysku do uwzględnienia |

## Tagi

Wszystkie taski są opatrzone tagiem `lsblk`, co pozwala na:

```bash
# Uruchomienie tylko części związanej z analizą dysków
ansible-playbook -i inventory playbook.yml --tags lsblk

# Pominięcie analizy dysków
ansible-playbook -i inventory playbook.yml --skip-tags lsblk
```

## Przykłady użycia

### Podstawowe użycie

```yaml
---
- name: Generate disk usage report
  hosts: all
  gather_facts: yes
  become: yes
  
  roles:
    - disk_usage_report
```

### Z konfiguracją zmiennych

```yaml
---
- name: Generate disk usage report with custom settings
  hosts: all
  gather_facts: yes
  become: yes
  
  vars:
    show_detailed_output: true
    check_unused_disks: true
    minimum_disk_size_gb: 5
  
  roles:
    - disk_usage_report
```

### Uruchomienie z tagami

```bash
# Uruchomienie całej roli
ansible-playbook -i inventory disk_usage_report_playbook.yml

# Uruchomienie tylko z tagiem lsblk
ansible-playbook -i inventory disk_usage_report_playbook.yml --tags lsblk

# Uruchomienie bez tagów lsblk
ansible-playbook -i inventory disk_usage_report_playbook.yml --skip-tags lsblk
```

## Wyjście

Rola generuje następujące raporty:

1. **Podsumowanie urządzeń blokowych** - lista wszystkich dysków z podstawowymi informacjami
2. **Raport nieużywanych dysków** - lista dysków bez partycji i punktów montowania (z pełnymi ścieżkami `/dev/`)
3. **Zamontowane systemy plików** - wyjście polecenia `df -h`
4. **Fakty podsumowujące** - strukturalne dane zapisane w `disk_usage_summary`

## Struktura faktów

```yaml
disk_usage_summary:
  hostname: "nazwa_hosta"
  total_devices: 3
  unused_disks_count: 1
  unused_disks:
    - name: "sdc"
      size: "500G"
      model: "Samsung SSD"
  unused_disks_paths:
    - "/dev/sdc"
  scan_date: "2025-07-09T10:30:00Z"
```

## Bezpieczeństwo

⚠️ **UWAGA**: Rola tylko identyfikuje nieużywane dyski, ale nie wykonuje na nich żadnych operacji. Zawsze sprawdź wyniki przed podjęciem jakichkolwiek działań na dyskach!

## Licencja

MIT
