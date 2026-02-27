# Plan de Ejecución - KIND en 146.190.60.216

## Servidor destino

| Campo   | Valor                |
|---------|----------------------|
| IP      | xxx.xxx.xxx.xxx       |
| Usuario | <user>                 |
| OS      | Ubuntu x86_64        |
| SSH     | Llave via 1Password  |

## Prerequisitos

1. Tener Ansible instalado en tu máquina local:
   ```bash
   # macOS
   brew install ansible

   # o con pip
   pip install ansible
   ```

2. Verificar que el agente SSH de 1Password esté activo:
   ```bash
   ssh toto@146.190.60.216 whoami
   ```

## Archivos

| Archivo           | Descripción                              |
|-------------------|------------------------------------------|
| `inventory.ini`   | Inventario con el servidor y usuario      |
| `kind-install.yml`| Playbook con toda la instalación          |

## Ejecución

### 1. Verificar conectividad con Ansible

```bash
cd "/Users/toto/Cursos/Cenfotec Kubernates"
ansible kind_servers -i inventory.ini -m ping
```

Resultado esperado:
```
146.190.60.216 | SUCCESS => { "ping": "pong" }
```

### 2. Ejecutar el playbook (modo prueba primero)

```bash
ansible-playbook -i inventory.ini kind-install.yml --check --ask-become-pass
```

Te pedirá la contraseña de sudo de `<user>`. Esto simula la ejecución sin hacer cambios reales. Revisa que no haya errores.

### 3. Ejecutar el playbook

```bash
ansible-playbook -i inventory.ini kind-install.yml --ask-become-pass
```

### 4. Verificar manualmente

Conéctate al servidor y confirma:

```bash
ssh admin@146.190.xx.xxx

# Verificar Docker
docker ps

# Verificar KIND
kind version

# Verificar kubectl
kubectl get nodes

# Verificar pods del sistema
kubectl get pods -n kube-system
```

## Qué instala el playbook (en orden)

1. **Docker** - Script oficial + agrega `toto` al grupo docker
2. **KIND v0.20.0** - Binario en `/usr/local/bin/kind`
3. **kubectl** - Via snap
4. **Cluster KIND "demo"** - 1 control-plane + 3 workers con puertos 80, 443, 30080, 6443
5. **Bash aliases** - `k`, `ka`, `kg` + autocompletado + kubectl-aliases

## Solución de problemas

| Problema | Solución |
|----------|----------|
| `Permission denied` en SSH | Verificar que 1Password CLI esté desbloqueado |
| `toto is not in the sudoers file` | Conectar como root y ejecutar `usermod -aG sudo toto` |
| El cluster no se crea | Revisar que Docker esté corriendo: `sudo systemctl status docker` |
| `snap not found` | Instalar snap: `sudo apt install snapd` |
