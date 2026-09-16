# Automatización de red — Ansible + EVE-NG (10 routers, malla parcial)

## 1. Instalar Ansible y la colección de Cisco en Ubuntu

```bash
sudo apt update
sudo apt install -y python3-pip
pip3 install ansible --break-system-packages
ansible-galaxy collection install -r requirements.yml
```

## 2. Bootstrap manual (una sola vez)

Antes de usar Ansible, cada router necesita SSH habilitado. Sigue
`00_bootstrap_manual.md` y aplica el bloque de configuración a los 10
routers por consola en EVE-NG.

## 3. Crear el vault con la contraseña de los routers

```bash
echo "mi-clave-secreta-del-vault" > .vault_pass
chmod 600 .vault_pass
ansible-vault create group_vars/vault.yml --vault-password-file .vault_pass
```

Dentro de `vault.yml` escribe:

```yaml
vault_admin_password: "Admin123!"
```

(usa la misma contraseña que pusiste en el bootstrap manual).

## 4. Probar conectividad

```bash
ansible cisco_routers -m ping --vault-password-file .vault_pass
```

Si todos responden `pong`, ya puedes automatizar.

## 5. Ejecutar todo el playbook maestro

```bash
ansible-playbook site.yml --vault-password-file .vault_pass
```

Esto, en orden:
1. Configura hostname, Loopback0 e interfaces IPv4/IPv6 en los 15 enlaces
2. Habilita OSPFv2 (IPv4) y OSPFv3 (IPv6) en área 0
3. Aplica seguridad: ACL de management en VTY, banner, cifrado de contraseñas
4. Descarga el running-config de cada router y lo cifra con ansible-vault

También puedes correr cada fase por separado:

```bash
ansible-playbook playbooks/01_interfaces.yml --vault-password-file .vault_pass
ansible-playbook playbooks/02_ospf.yml --vault-password-file .vault_pass
ansible-playbook playbooks/03_security.yml --vault-password-file .vault_pass
ansible-playbook playbooks/04_backup.yml --vault-password-file .vault_pass
```

## 6. Verificar

```bash
ansible cisco_routers -m ios_command -a "commands='show ip ospf neighbor'" --vault-password-file .vault_pass
ansible cisco_routers -m ios_command -a "commands='show ipv6 ospf neighbor'" --vault-password-file .vault_pass
```

## Estructura del proyecto

```
ansible-project/
├── ansible.cfg
├── inventory.yml              # 10 routers y su IP de management
├── requirements.yml           # colección cisco.ios
├── site.yml                   # playbook maestro
├── 00_bootstrap_manual.md     # paso manual único (habilitar SSH)
├── group_vars/
│   └── cisco_routers.yml      # credenciales, OSPF process-id/área
├── host_vars/
│   ├── R1.yml ... R10.yml     # hostname, loopback, interfaces por router
├── playbooks/
│   ├── 01_interfaces.yml
│   ├── 02_ospf.yml
│   ├── 03_security.yml
│   └── 04_backup.yml
└── backups/                   # se crea automáticamente, cifrado
```

## Direccionamiento

- Enlaces punto a punto: `10.0.<N>.0/30` (IPv4) y `2001:db8:<N>::/64` (IPv6), N = 1 a 15
- Loopback0: `10.255.255.<router>/32` y `2001:db8:ffff::<router>/128`
- Management: `192.168.100.0/24` (R1 = .11 ... R10 = .20)
