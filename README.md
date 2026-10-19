# Auto Infra

Automatización de seis routers Cisco L3 y seis switches Cisco L2 en EVE-NG.
El alcance usa IPv4, rutas estáticas, HSRP, SSH, ACLs y respaldos. No usa IPv6,
OSPF ni MSTP.

## Antes de ejecutar Ansible

1. Conectar una interfaz de gestión de cada equipo a la Cloud de EVE-NG.
2. Aplicar por consola el bootstrap de SSH y una IP de gestión a cada equipo.
3. Actualizar las IPs de gestión en `inventory.yml`.
4. Copiar `group_vars/vault.yml.example` como `group_vars/vault.yml` y reemplazar
   las claves de ejemplo.
5. Crear el archivo local `.vault_pass` con la contraseña elegida para Ansible Vault.
6. Instalar dependencias: `ansible-galaxy collection install -r requirements.yml`.

## Único punto de edición de la topología

Completar únicamente los archivos de `host_vars/`:

- Routers: `interfaces`, `static_routes` y `hsrp_groups`.
- Switches: `vlans`, `management_svi` y `switchports`.

Ejemplo para un router:

```yaml
interfaces:
  - name: GigabitEthernet0/1
    description: ENLACE_R1_A_R3
    address: 10.10.2.1
    mask: 255.255.255.252
static_routes:
  - network: 10.10.30.0
    mask: 255.255.255.0
    next_hop: 10.10.2.2
  - network: 10.10.40.0
    mask: 255.255.255.0
    next_hop: 10.10.2.2
    distance: 10
hsrp_groups:
  - interface: GigabitEthernet0/2.10
    group: 10
    virtual_ip: 10.10.10.1
    priority: 110
```

Ejemplo para un switch:

```yaml
vlans:
  - { id: 10, name: DIRECCION }
  - { id: 20, name: TI }
  - { id: 99, name: GESTION }
management_svi:
  vlan: 99
  address: 10.10.99.21
  mask: 255.255.255.0
  gateway: 10.10.99.1
switchports:
  - name: GigabitEthernet0/1
    lines:
      - description PC_DIRECCION
      - switchport mode access
      - switchport access vlan 10
      - spanning-tree portfast
  - name: GigabitEthernet0/2
    lines:
      - description TRUNK_A_SW2
      - switchport mode trunk
      - switchport trunk allowed vlan 10,20,99
```

## Ejecución

Primero validar SSH:

```bash
ansible-playbook 00_bootstrap.yml --vault-password-file .vault_pass
```

Después ejecutar por fases o todo el proyecto:

```bash
ansible-playbook 01_routers.yml --vault-password-file .vault_pass
ansible-playbook 02_switches.yml --vault-password-file .vault_pass
ansible-playbook 03_hsrp.yml --vault-password-file .vault_pass
ansible-playbook site.yml --vault-password-file .vault_pass
```
