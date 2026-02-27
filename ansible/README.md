# Ansible Playbook - KIND Kubernetes Cluster

Playbook de Ansible para aprovisionar un servidor Ubuntu con un cluster KIND (Kubernetes IN Docker).

## Servidor destino

| Campo   | Valor                          |
|---------|--------------------------------|
| IP      | Tu servidor Ubuntu             |
| Usuario | Usuario con acceso sudo        |
| OS      | Ubuntu 20.04+ (x86_64)         |
| SSH     | Llave SSH configurada          |

## Prerequisitos

### 1. Instalar Ansible en tu máquina local

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt install ansible

# Con pip
pip install ansible
```

### 2. Configurar el inventario

Edita `inventory.ini` y agrega tu servidor:

```ini
[kind_servers]
192.168.1.100 ansible_user=ubuntu
```

### 3. Verificar conexión SSH

```bash
ssh <usuario>@<ip_servidor> whoami
```

## Archivos

| Archivo           | Descripción                              |
|-------------------|------------------------------------------|
| `inventory.ini`   | Inventario con el servidor y usuario     |
| `kind-install.yml`| Playbook con toda la instalación         |

## Ejecución

### 1. Verificar conectividad con Ansible

```bash
ansible kind_servers -i inventory.ini -m ping
```

Resultado esperado:
```
<IP_SERVIDOR> | SUCCESS => { "ping": "pong" }
```

### 2. Ejecutar el playbook (modo prueba primero)

```bash
ansible-playbook -i inventory.ini kind-install.yml --check --ask-become-pass
```

Te pedirá la contraseña de sudo. Esto simula la ejecución sin hacer cambios reales.

### 3. Ejecutar el playbook

```bash
ansible-playbook -i inventory.ini kind-install.yml --ask-become-pass
```

### 4. Verificar manualmente

Conéctate al servidor y confirma:

```bash
ssh <usuario>@<ip_servidor>

# Verificar Docker
docker ps

# Verificar KIND
kind version

# Verificar kubectl
kubectl get nodes

# Verificar pods del sistema
kubectl get pods -n kube-system
```

## Qué instala el playbook

| # | Componente | Descripción |
|---|------------|-------------|
| 1 | **Docker** | Script oficial + agrega usuario al grupo docker |
| 2 | **KIND v0.20.0** | Binario en `/usr/local/bin/kind` |
| 3 | **kubectl** | Via snap |
| 4 | **Cluster KIND** | 1 control-plane + 3 workers |
| 5 | **zsh** | Con kubectl completion y aliases |
| 6 | **code-server** | VS Code en el navegador (puerto 3030) |

### Puertos expuestos del cluster

| Puerto | Uso |
|--------|-----|
| 80 | Ingress HTTP |
| 443 | Ingress HTTPS |
| 30080 | NodePort |
| 6443 | Kubernetes API |

## Solución de problemas

| Problema | Solución |
|----------|----------|
| `Permission denied` en SSH | Verificar que la llave SSH esté configurada |
| `user is not in the sudoers file` | Conectar como root y ejecutar `usermod -aG sudo <usuario>` |
| El cluster no se crea | Revisar que Docker esté corriendo: `sudo systemctl status docker` |
| `snap not found` | Instalar snap: `sudo apt install snapd` |

## Acceso a VS Code Server

Después de ejecutar el playbook, puedes acceder a VS Code en:

```
http://<IP_SERVIDOR>:3030
```
