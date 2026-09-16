# Bootstrap manual (una sola vez por router)

Ansible se conecta por SSH. Los routers recién levantados en EVE-NG no tienen
SSH ni usuario configurado, así que este paso inicial se hace a mano por
consola (o pegando el bloque completo en la consola de EVE-NG) — después de
esto, todo lo demás lo hace Ansible.

Conecta cada router a la red de management (nube/pnet0 en EVE-NG) por su
interfaz `GigabitEthernet0/0` y aplica esto, cambiando `HOSTNAME` y `MGMT_IP`
según la tabla de abajo:

```
enable
configure terminal
hostname HOSTNAME
ip domain-name lab.local
crypto key generate rsa modulus 2048
username admin privilege 15 secret Admin123!
line vty 0 4
 login local
 transport input ssh
exit
ip ssh version 2
interface GigabitEthernet0/0
 description MGMT
 ip address MGMT_IP 255.255.255.0
 no shutdown
exit
end
write memory
```

## Tabla de IPs de management (red 192.168.100.0/24)

| Router | Hostname | IP de management (G0/0) |
|--------|----------|--------------------------|
| R1  | R1  | 192.168.100.11 |
| R2  | R2  | 192.168.100.12 |
| R3  | R3  | 192.168.100.13 |
| R4  | R4  | 192.168.100.14 |
| R5  | R5  | 192.168.100.15 |
| R6  | R6  | 192.168.100.16 |
| R7  | R7  | 192.168.100.17 |
| R8  | R8  | 192.168.100.18 |
| R9  | R9  | 192.168.100.19 |
| R10 | R10 | 192.168.100.20 |

> Importante: cambia `Admin123!` por una contraseña real y guárdala también
> en `group_vars/vault.yml` (cifrado con ansible-vault) para que coincida
> con lo que Ansible usará para autenticarse.

## Verificación rápida

Desde la VM de Ubuntu donde corre Ansible:

```bash
ssh admin@192.168.100.11
```

Si conecta y pide la contraseña, el bootstrap fue exitoso. Repite para
los 10 routers antes de continuar con los playbooks.
