# FortiGate Configuracion Inicial - Ansible

Proyecto base para automatizar la configuracion inicial y tareas comunes de FortiGate con Ansible y la coleccion `fortinet.fortios`.

La intencion es servir como punto de partida publico para laboratorios, documentacion tecnica y automatizacion controlada de firewalls FortiGate. Incluye ejemplos seguros para:

- Backup de configuracion.
- Ajustes globales basicos.
- Objetos de firewall.
- Servicios personalizados.
- Rutas estaticas.
- Politicas IPv4.
- Estructura de inventarios y variables por ambiente.

> Este repositorio usa direcciones de documentacion como `192.0.2.10`. No incluye credenciales, configuraciones reales ni informacion de clientes.

## Estructura

```text
.
|-- ansible.cfg
|-- inventories/
|   `-- lab/
|       |-- hosts.yml
|       `-- group_vars/
|           `-- fortigates.yml
|-- playbooks/
|   |-- backup.yml
|   |-- baseline.yml
|   |-- firewall_objects.yml
|   |-- firewall_policies.yml
|   |-- site.yml
|   `-- static_routes.yml
|-- requirements.yml
|-- requirements.txt
`-- docs/
    `-- SECURITY.md
```

## Instalar WSL en Windows

Ansible funciona mejor desde Linux. Si trabajas desde un PC Windows, la forma recomendada es usar WSL con Ubuntu.

Abre PowerShell como administrador y ejecuta:

```powershell
wsl --install -d Ubuntu
```

Reinicia el PC si Windows lo solicita. Luego abre Ubuntu desde el menu inicio y actualiza paquetes:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y python3 python3-venv python3-pip git sshpass
```

Clona el proyecto dentro de WSL:

```bash
cd ~
git clone https://github.com/alejandrojx/FortiGate-Configuracion-Inicial-Ansible.git
cd FortiGate-Configuracion-Inicial-Ansible
```

Si ya descargaste el proyecto en Windows, tambien puedes entrar desde WSL a una ruta del PC:

```bash
cd /mnt/c/Users/TU_USUARIO/Documents/FortiGate-Configuracion-Inicial-Ansible
```

## Requisitos

- Python 3.10 o superior.
- Ansible Core.
- Acceso HTTPS/API al FortiGate.
- Usuario administrador o token API con permisos suficientes.
- WSL/Ubuntu si estas trabajando desde Windows.

Instalar dependencias desde WSL o Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

Ejemplo visto desde un PC con WSL:

```bash
alejandro@PC-WINDOWS:~$ cd ~/FortiGate-Configuracion-Inicial-Ansible
alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ python3 -m venv .venv
alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ source .venv/bin/activate
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ pip install -r requirements.txt
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ ansible-galaxy collection install -r requirements.yml
```

## Archivos que normalmente debes modificar

Para adaptar este proyecto a tu firewall o laboratorio, normalmente solo modificas estos archivos:

```text
inventories/lab/hosts.yml
inventories/lab/group_vars/fortigates.yml
```

En `inventories/lab/hosts.yml` cambias la IP o FQDN del FortiGate:

```yaml
all:
  children:
    fortigates:
      hosts:
        fgt-lab:
          ansible_host: 192.0.2.10
```

En `inventories/lab/group_vars/fortigates.yml` cambias hostname, VDOM, objetos, rutas y politicas:

```yaml
system_global:
  hostname: FGT-LAB
  timezone: 04
  admintimeout: 30

address_objects:
  - name: LAN_USERS
    subnet: 10.10.10.0 255.255.255.0
    comment: Example LAN users subnet
```

No guardes usuarios, passwords ni tokens reales en esos archivos. Usa variables de entorno o Ansible Vault.

## Configuracion Rapida

Define credenciales como variables de entorno dentro de WSL:

```bash
export FORTIGATE_USERNAME="admin"
export FORTIGATE_PASSWORD="change-me"
```

Ejemplo visto desde un PC:

```bash
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ export FORTIGATE_USERNAME="admin"
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ export FORTIGATE_PASSWORD="change-me"
```

Para validar que Ansible lee el inventario:

```bash
ansible-inventory --graph
```

Ejemplo de salida esperada:

```text
@all:
  |--@ungrouped:
  |--@fortigates:
  |  |--fgt-lab
```

## Uso

Todos los comandos se ejecutan desde la raiz del proyecto:

```bash
cd ~/FortiGate-Configuracion-Inicial-Ansible
source .venv/bin/activate
```

Probar conectividad contra el FortiGate:

```bash
ansible fortigates -m fortinet.fortios.fortios_monitor_fact -a "selector=system_status"
```

Ejemplo visto desde un PC:

```bash
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ ansible fortigates -m fortinet.fortios.fortios_monitor_fact -a "selector=system_status"
fgt-lab | SUCCESS => {
  "changed": false,
  "meta": {
    "version": "v7.x.x",
    "serial": "FGTXXXXXXXXXXXXX"
  }
}
```

Ejecutar backup:

```bash
ansible-playbook playbooks/backup.yml
```

Ejemplo visto desde un PC:

```bash
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ ansible-playbook playbooks/backup.yml

PLAY [Backup FortiGate configuration] *****************************************
TASK [Create local backup directory] ******************************************
changed: [fgt-lab -> localhost]
TASK [Download full configuration backup] *************************************
ok: [fgt-lab]
TASK [Save backup locally] ****************************************************
changed: [fgt-lab -> localhost]
```

El archivo quedaria en:

```text
backups/fgt-lab-YYYYMMDD-HHMMSS.conf
```

Aplicar configuracion inicial:

```bash
ansible-playbook playbooks/baseline.yml
```

Crear objetos de firewall:

```bash
ansible-playbook playbooks/firewall_objects.yml
```

Ejemplo de objeto definido en `inventories/lab/group_vars/fortigates.yml`:

```yaml
address_objects:
  - name: LAN_USERS
    subnet: 10.10.10.0 255.255.255.0
    comment: Example LAN users subnet
```

Ejemplo visto desde un PC:

```bash
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ ansible-playbook playbooks/firewall_objects.yml

PLAY [Configure FortiGate address and service objects] ************************
TASK [Configure firewall address objects] *************************************
changed: [fgt-lab] => (item={'name': 'LAN_USERS', 'subnet': '10.10.10.0 255.255.255.0', 'comment': 'Example LAN users subnet'})
TASK [Configure custom service objects] ***************************************
changed: [fgt-lab] => (item={'name': 'TCP_8443', 'tcp_portrange': '8443', 'comment': 'Example custom HTTPS service'})
```

Crear rutas estaticas:

```bash
ansible-playbook playbooks/static_routes.yml
```

Ejemplo de ruta definida en `inventories/lab/group_vars/fortigates.yml`:

```yaml
static_routes:
  - seq_num: 10
    dst: 0.0.0.0 0.0.0.0
    gateway: 192.0.2.1
    device: port1
    comment: Example default route
```

Crear politicas IPv4:

```bash
ansible-playbook playbooks/firewall_policies.yml
```

Ejemplo de politica definida en `inventories/lab/group_vars/fortigates.yml`:

```yaml
firewall_policies:
  - policyid: 100
    name: LAN_to_Internet
    srcintf: port2
    dstintf: port1
    srcaddr: LAN_USERS
    dstaddr: all
    service: ALL
    action: accept
    schedule: always
    nat: enable
    logtraffic: all
```

Aplicar todo:

```bash
ansible-playbook playbooks/site.yml
```

Ejemplo visto desde un PC:

```bash
(.venv) alejandro@PC-WINDOWS:~/FortiGate-Configuracion-Inicial-Ansible$ ansible-playbook playbooks/site.yml

PLAY [Apply FortiGate baseline settings] **************************************
PLAY [Configure FortiGate address and service objects] ************************
PLAY [Configure FortiGate static routes] **************************************
PLAY [Configure FortiGate firewall policies] **********************************
PLAY RECAP ********************************************************************
fgt-lab : ok=8 changed=4 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

## Buenas Practicas

- Ejecuta primero en laboratorio o en un VDOM de pruebas.
- Versiona cambios pequenos y revisables.
- Usa Ansible Vault o secretos de CI/CD para credenciales.
- Manten los backups fuera del repositorio.
- Revisa las variables antes de aplicar a produccion.
- Evita mezclar cambios manuales frecuentes con automatizacion sin documentarlos.

## Ansible vs Terraform para FortiGate

Para operacion diaria de FortiGate, Ansible suele ser mas flexible:

- Cambios por lotes.
- Playbooks por tarea.
- Plantillas y variables por cliente o sede.
- Backups y validaciones.
- Automatizacion incremental.

Terraform puede ser util cuando el objetivo es estado deseado estricto o despliegue repetible de infraestructura, especialmente en cloud. En entornos FortiGate ya existentes, Ansible suele ser mas comodo para automatizacion operativa.

## Seguridad

Lee [docs/SECURITY.md](docs/SECURITY.md) antes de adaptar este proyecto a un entorno real.

## Licencia

MIT. Puedes usar este proyecto como base para laboratorios, documentacion, formacion o automatizacion interna.
