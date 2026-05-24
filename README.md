# FortiGate Ansible Baseline

Proyecto base para automatizar configuraciones comunes de FortiGate con Ansible y la colección `fortinet.fortios`.

La intención es servir como punto de partida público para laboratorios, documentación técnica y automatización controlada de firewalls FortiGate. Incluye ejemplos seguros para:

- Backup de configuración.
- Ajustes globales básicos.
- Objetos de firewall.
- Servicios personalizados.
- Rutas estáticas.
- Políticas IPv4.
- Estructura de inventarios y variables por ambiente.

> Este repositorio usa direcciones de documentación como `192.0.2.10`. No incluye credenciales, configuraciones reales ni información de clientes.

## Estructura

```text
.
├── ansible.cfg
├── inventories/
│   └── lab/
│       ├── hosts.yml
│       └── group_vars/
│           └── fortigates.yml
├── playbooks/
│   ├── backup.yml
│   ├── baseline.yml
│   ├── firewall_objects.yml
│   ├── firewall_policies.yml
│   ├── site.yml
│   └── static_routes.yml
├── requirements.yml
├── requirements.txt
└── docs/
    └── SECURITY.md
```

## Requisitos

- Python 3.10 o superior.
- Ansible Core.
- Acceso HTTPS/API al FortiGate.
- Usuario administrador o token API con permisos suficientes.

Instalar dependencias:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

En Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

## Configuración Rápida

Define las credenciales como variables de entorno:

```bash
export FORTIGATE_USERNAME="admin"
export FORTIGATE_PASSWORD="change-me"
```

En PowerShell:

```powershell
$env:FORTIGATE_USERNAME = "admin"
$env:FORTIGATE_PASSWORD = "change-me"
```

Edita el inventario:

```yaml
all:
  children:
    fortigates:
      hosts:
        fgt-lab:
          ansible_host: 192.0.2.10
```

Edita las variables del laboratorio en:

```text
inventories/lab/group_vars/fortigates.yml
```

## Uso

Probar conectividad:

```bash
ansible fortigates -m fortinet.fortios.fortios_monitor_fact -a "selector=system_status" 
```

Ejecutar backup:

```bash
ansible-playbook playbooks/backup.yml
```

Aplicar solo baseline:

```bash
ansible-playbook playbooks/baseline.yml
```

Crear objetos:

```bash
ansible-playbook playbooks/firewall_objects.yml
```

Crear rutas:

```bash
ansible-playbook playbooks/static_routes.yml
```

Crear políticas:

```bash
ansible-playbook playbooks/firewall_policies.yml
```

Aplicar todo:

```bash
ansible-playbook playbooks/site.yml
```

## Buenas Prácticas

- Ejecuta primero en laboratorio o en un VDOM de pruebas.
- Versiona cambios pequeños y revisables.
- Usa Ansible Vault o secretos de CI/CD para credenciales.
- Mantén los backups fuera del repositorio.
- Revisa el diff y las variables antes de aplicar a producción.
- Evita mezclar cambios manuales frecuentes con automatización sin documentarlos.

## Ansible vs Terraform para FortiGate

Para operación diaria de FortiGate, Ansible suele ser más flexible:

- Cambios por lotes.
- Playbooks por tarea.
- Plantillas y variables por cliente o sede.
- Backups y validaciones.
- Automatización incremental.

Terraform puede ser útil cuando el objetivo es estado deseado estricto o despliegue repetible de infraestructura, especialmente en cloud. En entornos FortiGate ya existentes, Ansible suele ser más cómodo para automatización operativa.

## Seguridad

Lee [docs/SECURITY.md](docs/SECURITY.md) antes de adaptar este proyecto a un entorno real.

## Licencia

MIT. Puedes usar este proyecto como base para laboratorios, documentación, formación o automatización interna.

