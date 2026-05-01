# ansible-updates

Zentrale Update-Steuerung für Proxmox-VMs via Ansible / Semaphore.

## Struktur

```
ansible-updates/
├── ansible.cfg
├── inventory/
│   ├── hosts.yml              # Hosts und Gruppen
│   └── group_vars/
│       └── all.yml            # Globale Defaults (SSH-User, Reboot-Policy, ...)
└── playbooks/
    ├── update.yml             # Pakete updaten (apt + dnf), optional Reboot
    ├── check-updates.yml      # Nur prüfen (Dry-Run-Report)
    └── reboot-if-needed.yml   # Reboot nur wenn nötig
```

## Inventory pflegen

Hosts in [inventory/hosts.yml](inventory/hosts.yml) eintragen — jeweils:

1. unter einer **Umgebungs-Gruppe** (`test`, `prod`, `proxmox`),
2. unter einer **OS-Familien-Gruppe** (`debian_family` oder `redhat_family`).

Beispiel:

```yaml
prod:
  hosts:
    web01:
      ansible_host: 192.168.1.20
    db01:
      ansible_host: 192.168.1.21
      ansible_user: ansible    # überschreibt Default

debian_family:
  hosts:
    web01: {}
    db01: {}
```

## Verhalten konfigurieren

In [inventory/group_vars/all.yml](inventory/group_vars/all.yml):

| Variable | Werte | Default |
|---|---|---|
| `ansible_user` | beliebig | `root` |
| `update_reboot_policy` | `never` / `if_needed` / `always` | `if_needed` |
| `update_reboot_timeout` | Sekunden | `600` |
| `apt_upgrade_type` | `safe` / `full` / `dist` | `safe` |
| `dnf_upgrade_type` | `default` / `minimal` / `security` | `default` |

Pro Gruppe überschreibbar via `inventory/group_vars/<gruppe>.yml`,
pro Host direkt im Inventory.

## Lokal testen (ohne Semaphore)

```bash
cd /root/ansible-updates

# Verbindungs-Test
ansible all -m ping

# Nur prüfen welche Updates anstehen
ansible-playbook playbooks/check-updates.yml

# Updates auf Test-Gruppe rollen
ansible-playbook playbooks/update.yml -e target=test

# Updates auf einen einzelnen Host
ansible-playbook playbooks/update.yml -e target=web01

# Mit anderer Reboot-Policy überschreiben
ansible-playbook playbooks/update.yml -e target=prod -e update_reboot_policy=never
```

## Einbindung in Semaphore

Siehe Abschnitt "Semaphore-Setup" — das Repo wird in Semaphore unter
**Project → Repositories** verlinkt, dann **Inventory** und **Templates**
(= Jobs) angelegt.
