# GUIA COMPLETA DE KUBERNETES - FOR DUMMIES

## Curso Cenfotec - Profesor Carlos Chacon Calvo

---

# TABLA DE CONTENIDOS

1. [Introduccion y Conceptos Basicos](#1-introduccion-y-conceptos-basicos)
2. [Instalacion del Ambiente](#2-instalacion-del-ambiente)
3. [Anatomia de un Archivo YAML](#3-anatomia-de-un-archivo-yaml)
4. [Pods - La Unidad Basica](#4-pods---la-unidad-basica)
5. [ReplicaSets - Garantizando Disponibilidad](#5-replicasets---garantizando-disponibilidad)
6. [Deployments - El Estandar de Produccion](#6-deployments---el-estandar-de-produccion)
7. [DaemonSets - Un Pod por Nodo](#7-daemonsets---un-pod-por-nodo)
8. [Namespaces - Organizando el Cluster](#8-namespaces---organizando-el-cluster)
9. [Services - Exponiendo Aplicaciones](#9-services---exponiendo-aplicaciones)
10. [Networking e Ingress](#10-networking-e-ingress)
11. [Storage - Persistencia de Datos](#11-storage---persistencia-de-datos)
12. [ConfigMaps y Secrets](#12-configmaps-y-secrets)
13. [Lifecycle - Ciclo de Vida](#13-lifecycle---ciclo-de-vida)
14. [Cuotas y Limites](#14-cuotas-y-limites)
15. [Taints y Tolerations](#15-taints-y-tolerations)
16. [Comandos CLI Esenciales](#16-comandos-cli-esenciales)
17. [Mapa de Relaciones entre Recursos](#17-mapa-de-relaciones-entre-recursos)
18. [Proyecto Final - Arquitectura de Microservicios](#18-proyecto-final---arquitectura-de-microservicios)

---

# 1. INTRODUCCION Y CONCEPTOS BASICOS

## Que es Kubernetes?

Kubernetes (K8s) es un **orquestador de contenedores**. Imagina que tienes muchos contenedores Docker corriendo - Kubernetes se encarga de:

- **Decidir donde corren** los contenedores
- **Reiniciarlos** si fallan
- **Escalarlos** si hay mas demanda
- **Balancear el trafico** entre ellos
- **Actualizar** las aplicaciones sin downtime

## Arquitectura del Cluster

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLUSTER DE KUBERNETES                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    CONTROL PLANE (Master)                    │   │
│  │  ┌───────────────┐ ┌──────────────┐ ┌────────────────────┐  │   │
│  │  │ kube-apiserver│ │    etcd      │ │  kube-scheduler    │  │   │
│  │  │               │ │  (Base de   │ │  (Decide donde     │  │   │
│  │  │ (Punto de    │ │   datos)     │ │   corren los pods) │  │   │
│  │  │  entrada)     │ │              │ │                    │  │   │
│  │  └───────────────┘ └──────────────┘ └────────────────────┘  │   │
│  │  ┌────────────────────────────────────────────────────────┐ │   │
│  │  │            kube-controller-manager                      │ │   │
│  │  │  (Ejecuta ReplicaSet, Deployment, etc.)                │ │   │
│  │  └────────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐   │
│  │   WORKER NODE 1  │ │   WORKER NODE 2  │ │   WORKER NODE 3  │   │
│  │  ┌────────────┐  │ │  ┌────────────┐  │ │  ┌────────────┐  │   │
│  │  │  kubelet   │  │ │  │  kubelet   │  │ │  │  kubelet   │  │   │
│  │  │ (Agente)   │  │ │  │ (Agente)   │  │ │  │ (Agente)   │  │   │
│  │  └────────────┘  │ │  └────────────┘  │ │  └────────────┘  │   │
│  │  ┌────────────┐  │ │  ┌────────────┐  │ │  ┌────────────┐  │   │
│  │  │ kube-proxy │  │ │  │ kube-proxy │  │ │  │ kube-proxy │  │   │
│  │  │  (Red)     │  │ │  │  (Red)     │  │ │  │  (Red)     │  │   │
│  │  └────────────┘  │ │  └────────────┘  │ │  └────────────┘  │   │
│  │  ┌────────────┐  │ │  ┌────────────┐  │ │  ┌────────────┐  │   │
│  │  │   PODs     │  │ │  │   PODs     │  │ │  │   PODs     │  │   │
│  │  │ ┌──┐ ┌──┐  │  │ │  │ ┌──┐ ┌──┐  │  │ │  │ ┌──┐ ┌──┐  │  │   │
│  │  │ │C1│ │C2│  │  │ │  │ │C1│ │C2│  │  │ │  │ │C1│ │C2│  │  │   │
│  │  │ └──┘ └──┘  │  │ │  │ └──┘ └──┘  │  │ │  │ └──┘ └──┘  │  │   │
│  │  └────────────┘  │ │  └────────────┘  │ │  └────────────┘  │   │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## Componentes Explicados

| Componente | Que hace | Analogia |
|------------|----------|----------|
| **kube-apiserver** | Recibe todas las peticiones (kubectl, UI, etc.) | La recepcion de un hotel |
| **etcd** | Guarda TODO el estado del cluster | El archivo/base de datos |
| **kube-scheduler** | Decide en que nodo poner cada Pod | El asignador de habitaciones |
| **kube-controller-manager** | Mantiene el estado deseado | El supervisor que verifica todo |
| **kubelet** | Agente en cada nodo que ejecuta Pods | El conserje de cada piso |
| **kube-proxy** | Maneja la red en cada nodo | El sistema de telefonia |

---

# 2. INSTALACION DEL AMBIENTE

## Opcion 1: KIND (Kubernetes IN Docker) - RECOMENDADA

KIND usa contenedores Docker para simular nodos de Kubernetes.

### Paso 1: Instalar Docker
```bash
sudo apt update
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

### Paso 2: Instalar Kind
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

### Paso 3: Instalar kubectl
```bash
sudo snap install kubectl --classic
```

### Paso 4: Crear archivo de configuracion (kind-cluster.yaml)

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: demo

nodes:
  - role: control-plane           # Nodo maestro
    kubeadmConfigPatches:
      - |
        kind: ClusterConfiguration
        apiServer:
          certSANs:
          - "PUBLIC_IP"           # Tu IP publica
          - "localhost"
          - "127.0.0.1"
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
    extraPortMappings:            # IMPORTANTE: Mapeo de puertos
      - containerPort: 80         # Para Ingress HTTP
        hostPort: 80
        protocol: TCP
      - containerPort: 443        # Para Ingress HTTPS
        hostPort: 443
        protocol: TCP
      - containerPort: 30080      # Para NodePort
        hostPort: 30080
        protocol: TCP
      - containerPort: 6443       # Para API de Kubernetes
        hostPort: 6443
        listenAddress: "0.0.0.0"
        protocol: TCP

  - role: worker                  # Nodo trabajador 1
  - role: worker                  # Nodo trabajador 2
  - role: worker                  # Nodo trabajador 3
```

### Explicacion de extraPortMappings

```
┌─────────────────────────────────────────────────────────────────┐
│                        TU COMPUTADORA                           │
│                                                                 │
│   Puerto 80 ─────────► containerPort: 80  (Ingress HTTP)       │
│   Puerto 443 ────────► containerPort: 443 (Ingress HTTPS)      │
│   Puerto 30080 ──────► containerPort: 30080 (NodePort)         │
│   Puerto 6443 ───────► containerPort: 6443 (API K8s)           │
│                                                                 │
│              ┌──────────────────────────────────┐               │
│              │   CONTENEDOR DOCKER (KIND)       │               │
│              │   ┌────────────────────────┐     │               │
│              │   │   CLUSTER KUBERNETES   │     │               │
│              │   │                        │     │               │
│              │   │   Control Plane + 3    │     │               │
│              │   │   Worker Nodes         │     │               │
│              │   └────────────────────────┘     │               │
│              └──────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

### Paso 5: Crear el cluster
```bash
kind create cluster --config kind-cluster.yaml
kubectl cluster-info --context kind-demo
```

### Verificar instalacion
```bash
kubectl get nodes
```

Resultado esperado:
```
NAME                 STATUS   ROLES           AGE     VERSION
demo-control-plane   Ready    control-plane   6m40s   v1.27.3
demo-worker          Ready    <none>          6m12s   v1.27.3
demo-worker2         Ready    <none>          6m12s   v1.27.3
demo-worker3         Ready    <none>          6m13s   v1.27.3
```

---

# 3. ANATOMIA DE UN ARCHIVO YAML

Todo en Kubernetes se define con archivos YAML. Es FUNDAMENTAL entender su estructura.

## Estructura Base de TODO Objeto Kubernetes

```yaml
apiVersion: v1                    # 1. VERSION DEL API
kind: Pod                         # 2. TIPO DE OBJETO
metadata:                         # 3. METADATOS
  name: mi-pod                    #    - Nombre unico
  namespace: default              #    - Namespace (opcional)
  labels:                         #    - Etiquetas (muy importantes!)
    app: mi-app
    env: produccion
spec:                             # 4. ESPECIFICACION
  # ... depende del tipo de objeto
```

## Los 4 Campos Obligatorios

### 1. apiVersion - Version del API

| Objeto | apiVersion |
|--------|------------|
| Pod, Service, ConfigMap, Secret, Namespace | `v1` |
| Deployment, ReplicaSet, DaemonSet | `apps/v1` |
| Ingress | `networking.k8s.io/v1` |
| PersistentVolume, PersistentVolumeClaim | `v1` |
| ResourceQuota, LimitRange | `v1` |

**Por que diferentes versiones?**
- `v1` = API "core" (objetos basicos)
- `apps/v1` = API de aplicaciones (workloads)
- `networking.k8s.io/v1` = API de networking

### 2. kind - Tipo de Objeto

Define QUE estas creando:
- `Pod`, `Deployment`, `ReplicaSet`, `DaemonSet`
- `Service`, `Ingress`
- `ConfigMap`, `Secret`
- `PersistentVolume`, `PersistentVolumeClaim`
- `Namespace`, `ResourceQuota`, `LimitRange`

### 3. metadata - Metadatos

```yaml
metadata:
  name: mi-app              # Nombre unico en el namespace
  namespace: produccion     # En que namespace vive (default: default)
  labels:                   # Etiquetas para organizar/seleccionar
    app: mi-app
    version: "1.0"
    team: backend
  annotations:              # Informacion adicional (no para seleccion)
    description: "API principal"
```

### 4. spec - Especificacion

Es diferente para cada tipo de objeto. Aqui defines el "estado deseado".

---

# 4. PODS - LA UNIDAD BASICA

## Que es un Pod?

Un **Pod** es la unidad mas pequena en Kubernetes. Es un "envoltorio" alrededor de uno o mas contenedores.

```
┌─────────────────────────────────────────────┐
│                    POD                       │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │   Contenedor 1  │  │   Contenedor 2  │   │
│  │   (nginx)       │  │   (sidecar)     │   │
│  │                 │  │                 │   │
│  └─────────────────┘  └─────────────────┘   │
│                                             │
│  Comparten:                                 │
│  - RED (mismo IP)                           │
│  - VOLUMENES                                │
│  - Se escalan JUNTOS                        │
│                                             │
│  IP: 10.244.1.5                             │
└─────────────────────────────────────────────┘
```

## Estructura de un Pod

```yaml
apiVersion: v1                    # API core
kind: Pod                         # Tipo: Pod
metadata:
  name: myapp-pod                 # Nombre unico
  labels:                         # ETIQUETAS (muy importantes!)
    app: myapp
    type: frontend
spec:
  containers:                     # Lista de contenedores
    - name: nginx-container       # Nombre del contenedor
      image: nginx                # Imagen de Docker
      ports:                      # Puertos (documentativo)
        - containerPort: 80
```

## Campos del Pod Explicados

### Labels (Etiquetas)

Las **labels** son pares clave-valor que identifican objetos:

```yaml
labels:
  app: myapp          # Nombre de la aplicacion
  type: frontend      # Tipo de componente
  env: production     # Ambiente
  version: "2.0"      # Version
```

**Por que son importantes?**
- Los **Services** encuentran Pods por labels
- Los **ReplicaSets** gestionan Pods por labels
- Puedes filtrar recursos con `kubectl get pods -l app=myapp`

### Contenedores

```yaml
containers:
  - name: nginx-container         # Nombre (obligatorio)
    image: nginx:1.21             # Imagen (obligatorio)
    ports:                        # Puertos (opcional, documentativo)
      - containerPort: 80
    env:                          # Variables de entorno
      - name: MI_VARIABLE
        value: "mi-valor"
    resources:                    # Recursos (muy recomendado)
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "250m"
```

## Por que falla un Pod con 2 contenedores nginx?

```yaml
# ESTO FALLA!
containers:
  - name: nginx-1
    image: nginx         # Puerto 80
  - name: nginx-2
    image: nginx         # Puerto 80 - CONFLICTO!
```

Los contenedores en un Pod comparten la misma red (mismo IP). Ambos nginx intentan usar el puerto 80 = conflicto.

**Soluciones:**
1. Usar diferentes puertos
2. Usar Pods separados

## Comandos para Pods

```bash
# Crear Pod
kubectl apply -f pod.yaml

# Listar Pods
kubectl get pods
kubectl get pods -o wide              # Mas columnas
kubectl get pods --show-labels        # Ver etiquetas

# Filtrar por etiqueta
kubectl get pods -l app=myapp

# Ver detalles
kubectl describe pod myapp-pod

# Ver logs
kubectl logs myapp-pod
kubectl logs myapp-pod -c nginx-container   # Contenedor especifico

# Ejecutar comando dentro del Pod
kubectl exec -it myapp-pod -- /bin/sh

# Eliminar
kubectl delete pod myapp-pod
kubectl delete -f pod.yaml
```

---

# 5. REPLICASETS - GARANTIZANDO DISPONIBILIDAD

## Que es un ReplicaSet?

Un **ReplicaSet** garantiza que un numero especifico de Pods identicos esten corriendo.

```
┌─────────────────────────────────────────────────────────────────┐
│                       REPLICASET                                │
│                                                                 │
│  Estado Deseado: replicas: 4                                   │
│                                                                 │
│  ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐                        │
│  │ Pod │   │ Pod │   │ Pod │   │ Pod │                        │
│  │  1  │   │  2  │   │  3  │   │  4  │                        │
│  └─────┘   └─────┘   └─────┘   └─────┘                        │
│                                                                 │
│  Si Pod 3 muere ──────────────────┐                           │
│                                    │                           │
│  ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐                        │
│  │ Pod │   │ Pod │   │ Pod │   │ Pod │  <── ReplicaSet crea   │
│  │  1  │   │  2  │   │ NEW │   │  4  │      uno nuevo         │
│  └─────┘   └─────┘   └─────┘   └─────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

## Estructura de un ReplicaSet

```yaml
apiVersion: apps/v1               # API de aplicaciones
kind: ReplicaSet
metadata:
  name: frontend-rs
  labels:
    app: myapp
    type: frontend
spec:
  replicas: 4                     # Cuantos Pods quiero

  selector:                       # Como ENCUENTRO mis Pods
    matchLabels:
      type: frontend              # DEBE coincidir con template.labels

  template:                       # Template para crear Pods
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: frontend            # DEBE coincidir con selector
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

## Relacion Selector - Template Labels

```
┌──────────────────────────────────────────────────────────────────┐
│                         REPLICASET                               │
│                                                                  │
│  spec:                                                           │
│    selector:                                                     │
│      matchLabels:           ◄────────┐                          │
│        type: frontend               │ DEBEN                     │
│                                     │ COINCIDIR                 │
│    template:                        │                           │
│      metadata:                      │                           │
│        labels:              ◄───────┘                           │
│          type: frontend                                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**ERROR COMUN:** Si no coinciden, el ReplicaSet no puede "encontrar" sus Pods.

```
Error: `selector` does not match template `labels`
```

## Comandos para ReplicaSets

```bash
# Crear
kubectl apply -f rs.yaml

# Listar
kubectl get replicaset
kubectl get rs              # Alias

# Escalar
kubectl scale --replicas=4 replicaset frontend-rs

# Ver detalles
kubectl describe rs frontend-rs

# Eliminar Pods (el RS los recrea!)
kubectl delete pods -l type=frontend
```

---

# 6. DEPLOYMENTS - EL ESTANDAR DE PRODUCCION

## Que es un Deployment?

Un **Deployment** es una capa sobre ReplicaSet que agrega:
- Actualizaciones controladas (Rolling Update)
- Historial de versiones
- Rollback facil

## Jerarquia de Objetos

```
┌─────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT                                │
│  - Maneja actualizaciones                                       │
│  - Guarda historial                                             │
│  - Permite rollback                                             │
│                                                                 │
│    │                                                            │
│    │ crea y gestiona                                           │
│    ▼                                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    REPLICASET                            │   │
│  │  - Mantiene N replicas                                   │   │
│  │                                                          │   │
│  │    │                                                     │   │
│  │    │ crea y gestiona                                    │   │
│  │    ▼                                                     │   │
│  │  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                    │   │
│  │  │ Pod │  │ Pod │  │ Pod │  │ Pod │                    │   │
│  │  │     │  │     │  │     │  │     │                    │   │
│  │  │┌───┐│  │┌───┐│  │┌───┐│  │┌───┐│                    │   │
│  │  ││ C ││  ││ C ││  ││ C ││  ││ C ││  ◄── Contenedores │   │
│  │  │└───┘│  │└───┘│  │└───┘│  │└───┘│                    │   │
│  │  └─────┘  └─────┘  └─────┘  └─────┘                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Estructura de un Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myfrontend-deployment
  labels:
    app: mywebsite
    type: frontend
spec:
  replicas: 4

  selector:                       # Como encuentro mis Pods
    matchLabels:
      app: mywebsite
      type: frontend

  template:                       # Template del Pod
    metadata:
      name: myapp-pod
      labels:
        app: mywebsite
        type: frontend
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

## Estrategias de Actualizacion

### RollingUpdate (Default)

Actualiza Pods gradualmente sin downtime.

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%          # Cuantos Pods EXTRA durante update
      maxUnavailable: 25%    # Cuantos pueden estar DOWN
```

```
Replicas: 4, maxSurge: 25%, maxUnavailable: 25%

Estado inicial:     [v1] [v1] [v1] [v1]
Paso 1:            [v1] [v1] [v1] [v1] [v2]  <-- crea 1 nuevo
Paso 2:            [v1] [v1] [v1] [v2] [v2]  <-- mata 1 viejo, crea 1 nuevo
...
Final:             [v2] [v2] [v2] [v2]       <-- todos actualizados
```

### Recreate

Mata TODOS los Pods y luego crea los nuevos. HAY DOWNTIME.

```yaml
spec:
  strategy:
    type: Recreate
```

```
Estado inicial:     [v1] [v1] [v1] [v1]
Paso 1:            [--] [--] [--] [--]  <-- DOWNTIME
Paso 2:            [v2] [v2] [v2] [v2]
```

## Comandos para Deployments

```bash
# Crear
kubectl apply -f deployment.yaml

# Listar
kubectl get deployment
kubectl get deployment,rs,pods      # Ver jerarquia completa

# Actualizar imagen
kubectl set image deployment/myfrontend nginx=nginx:1.21

# Reiniciar Pods
kubectl rollout restart deployment myfrontend

# Ver historial
kubectl rollout history deployment myfrontend

# Rollback
kubectl rollout undo deployment myfrontend

# Escalar
kubectl scale deployment myfrontend --replicas=6
```

---

# 7. DAEMONSETS - UN POD POR NODO

## Que es un DaemonSet?

Un **DaemonSet** asegura que TODOS los nodos (o un subconjunto) ejecuten una copia del Pod.

```
┌─────────────────────────────────────────────────────────────────┐
│                        DAEMONSET                                 │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   NODE 1     │  │   NODE 2     │  │   NODE 3     │          │
│  │  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │          │
│  │  │  Pod   │  │  │  │  Pod   │  │  │  │  Pod   │  │          │
│  │  │(agente)│  │  │  │(agente)│  │  │  │(agente)│  │          │
│  │  └────────┘  │  │  └────────┘  │  │  └────────┘  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                 │
│  Si agregas NODE 4 ──► Automaticamente crea Pod en NODE 4      │
└─────────────────────────────────────────────────────────────────┘
```

## Casos de Uso

- **Agentes de monitoreo** (Prometheus Node Exporter)
- **Colectores de logs** (Fluentd, Filebeat)
- **Plugins de red** (kube-proxy, CNI)
- **Agentes de seguridad**

## Estructura de un DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-demo
  labels:
    app: ds-demo
spec:
  selector:
    matchLabels:
      app: ds-demo
  template:
    metadata:
      labels:
        app: ds-demo
    spec:
      containers:
      - name: ds-demo
        image: nginx
```

## DaemonSet vs Deployment

| Caracteristica | Deployment | DaemonSet |
|---------------|------------|-----------|
| Replicas | N configurables | 1 por nodo |
| Escalado | Manual o HPA | Automatico con nodos |
| Uso tipico | Aplicaciones | Infraestructura/Agentes |
| Control Plane | NO (por defecto) | Depende de tolerations |

## Comandos

```bash
# Listar DaemonSets
kubectl get ds
kubectl get ds -n kube-system    # Ver los de sistema

# Ver detalles
kubectl describe ds ds-demo
```

---

# 8. NAMESPACES - ORGANIZANDO EL CLUSTER

## Que es un Namespace?

Un **Namespace** es una division logica del cluster para separar recursos.

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLUSTER                                   │
│                                                                 │
│  ┌───────────────────────┐  ┌───────────────────────┐          │
│  │    NAMESPACE: dev     │  │   NAMESPACE: prod     │          │
│  │                       │  │                       │          │
│  │  - Pods de desarrollo │  │  - Pods de produccion │          │
│  │  - ConfigMaps dev     │  │  - ConfigMaps prod    │          │
│  │  - Secrets dev        │  │  - Secrets prod       │          │
│  │  - Cuotas limitadas   │  │  - Mas recursos       │          │
│  │                       │  │                       │          │
│  └───────────────────────┘  └───────────────────────┘          │
│                                                                 │
│  ┌───────────────────────┐  ┌───────────────────────┐          │
│  │  NAMESPACE: team-a    │  │  NAMESPACE: team-b    │          │
│  │                       │  │                       │          │
│  │  - Recursos equipo A  │  │  - Recursos equipo B  │          │
│  └───────────────────────┘  └───────────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

## Namespaces por Defecto

| Namespace | Descripcion |
|-----------|-------------|
| `default` | Donde van recursos si no especificas namespace |
| `kube-system` | Componentes del sistema (etcd, scheduler, etc.) |
| `kube-public` | Recursos publicos (poco usado) |
| `kube-node-lease` | Para health checks de nodos |

## Crear un Namespace

### Via YAML
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### Via CLI
```bash
kubectl create namespace dev
```

## Asignar Recursos a un Namespace

### En el YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod
  namespace: dev          # <-- Aqui
spec:
  containers:
    - name: nginx
      image: nginx
```

### Con el comando
```bash
kubectl run redis-pod --image=redis -n dev
```

## DNS Interno entre Namespaces

Kubernetes crea DNS automatico para servicios:

```
<servicio>.<namespace>.svc.cluster.local
```

Ejemplo:
```
# Desde el namespace "frontend", llamar al servicio "api" en namespace "backend"
http://api.backend.svc.cluster.local:8080
```

## Comandos

```bash
# Listar namespaces
kubectl get ns

# Ver recursos de un namespace
kubectl get pods -n dev
kubectl get all -n dev

# Ver todos los namespaces
kubectl get pods -A
kubectl get all -A

# Cambiar namespace por defecto
kubectl config set-context --current --namespace=dev

# O usar kubens (herramienta externa)
kubens dev
```

---

# 9. SERVICES - EXPONIENDO APLICACIONES

## Que es un Service?

Un **Service** es una abstraccion que define una forma de acceder a un conjunto de Pods.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│    CLIENTE                                                      │
│       │                                                         │
│       │ IP: 10.96.0.1:80                                       │
│       ▼                                                         │
│  ┌─────────────────────────────────────────┐                   │
│  │              SERVICE                     │                   │
│  │  - IP estable                           │                   │
│  │  - Balancea trafico                     │                   │
│  │  - Encuentra Pods por LABELS            │                   │
│  └─────────────────────────────────────────┘                   │
│       │                                                         │
│       │ selector: app=myapp                                    │
│       ▼                                                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                    │
│  │   Pod   │    │   Pod   │    │   Pod   │                    │
│  │ app=    │    │ app=    │    │ app=    │                    │
│  │ myapp   │    │ myapp   │    │ myapp   │                    │
│  │         │    │         │    │         │                    │
│  │ :8080   │    │ :8080   │    │ :8080   │                    │
│  └─────────┘    └─────────┘    └─────────┘                    │
│                                                                 │
│  ENDPOINTS: 10.244.1.5:8080, 10.244.2.3:8080, 10.244.3.7:8080 │
└─────────────────────────────────────────────────────────────────┘
```

## Tipos de Services

### 1. ClusterIP (Default)

IP interna solo accesible DENTRO del cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  type: ClusterIP           # Default, puede omitirse
  selector:
    app: myapp              # Busca Pods con label app=myapp
  ports:
    - port: 80              # Puerto del Service
      targetPort: 8080      # Puerto del contenedor
```

### 2. NodePort

Abre un puerto (30000-32767) en TODOS los nodos.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80              # Puerto del Service (interno)
      targetPort: 8080      # Puerto del contenedor
      nodePort: 30080       # Puerto en cada nodo (externo)
```

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   INTERNET                                                      │
│       │                                                         │
│       │ http://NODE_IP:30080                                   │
│       ▼                                                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                    │
│  │ NODE 1  │    │ NODE 2  │    │ NODE 3  │                    │
│  │ :30080  │    │ :30080  │    │ :30080  │                    │
│  └────┬────┘    └────┬────┘    └────┬────┘                    │
│       │              │              │                          │
│       └──────────────┼──────────────┘                          │
│                      ▼                                         │
│              ┌───────────────┐                                 │
│              │    SERVICE    │                                 │
│              │   ClusterIP   │                                 │
│              │    :80        │                                 │
│              └───────┬───────┘                                 │
│                      ▼                                         │
│              ┌───────────────┐                                 │
│              │     PODs      │                                 │
│              │    :8080      │                                 │
│              └───────────────┘                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3. LoadBalancer

Para nubes (AWS, GCP, Azure). Crea un Load Balancer externo con IP publica.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

## Entendiendo los Puertos

```yaml
ports:
  - targetPort: 8080     # Puerto del CONTENEDOR (donde escucha la app)
    port: 80             # Puerto del SERVICE (como otros Pods lo llaman)
    nodePort: 30080      # Puerto del NODO (acceso externo)
```

```
EXTERNO ──► nodePort:30080 ──► port:80 ──► targetPort:8080 ──► Contenedor
```

## Endpoints

Los **Endpoints** son las IPs de los Pods que el Service encontro usando el selector.

```bash
kubectl get endpoints mi-servicio
```

---

# 10. NETWORKING E INGRESS

## Modelo de Red de Kubernetes

- Cada Pod tiene su propia IP
- Pods pueden comunicarse entre si sin NAT
- Nodos pueden comunicarse con Pods sin NAT

## Que es un Ingress?

Un **Ingress** maneja acceso HTTP/HTTPS externo a Services dentro del cluster.

```
┌─────────────────────────────────────────────────────────────────┐
│                         INTERNET                                 │
│                             │                                    │
│      ┌──────────────────────┼──────────────────────┐            │
│      │                      ▼                      │            │
│      │            ┌─────────────────┐             │            │
│      │            │ INGRESS CONTROLLER │            │            │
│      │            │  (nginx, traefik)  │            │            │
│      │            └─────────┬─────────┘            │            │
│      │                      │                      │            │
│      │    ┌─────────────────┼─────────────────┐   │            │
│      │    │                 │                 │   │            │
│      │    ▼                 ▼                 ▼   │            │
│      │  api.com/          web.com/       admin.com/            │
│      │    │                 │                 │   │            │
│      │    ▼                 ▼                 ▼   │            │
│      │ ┌──────┐         ┌──────┐         ┌──────┐│            │
│      │ │ SVC  │         │ SVC  │         │ SVC  ││            │
│      │ │ api  │         │ web  │         │admin ││            │
│      │ └──┬───┘         └──┬───┘         └──┬───┘│            │
│      │    │                │                │    │            │
│      │    ▼                ▼                ▼    │            │
│      │  PODs              PODs              PODs  │            │
│      │                                           │            │
│      └───────────────────────────────────────────┘            │
│                        CLUSTER                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Componentes Necesarios

1. **Ingress Controller** - El "motor" que implementa las reglas (nginx-ingress, traefik)
2. **Ingress Resource** - Las reglas de enrutamiento (YAML)
3. **Service** - El destino final

## Instalar Ingress Controller (KIND)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Estructura de un Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: http-ingress
spec:
  rules:
  - host: miapp.kubelabs.dev          # Dominio
    http:
      paths:
      - path: /                        # Ruta
        pathType: Prefix
        backend:
          service:
            name: mi-servicio          # Nombre del Service
            port:
              number: 80               # Puerto del Service
```

## Ejemplo Completo

### 1. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
        ports:
        - containerPort: 80
```

### 2. Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

### 3. Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: web.midominio.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

## Simular Dominios Localmente

Editar `/etc/hosts`:
```
127.0.0.1  web.midominio.com
127.0.0.1  api.midominio.com
```

---

# 11. STORAGE - PERSISTENCIA DE DATOS

## El Problema

Los contenedores son **efimeros**. Si un Pod muere, sus datos desaparecen.

## La Solucion: Persistent Volumes

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  StorageClass (opcional)                                       │
│  - Define COMO se crea el almacenamiento                       │
│  - Dynamic provisioning                                        │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────────┐                                       │
│  │  PersistentVolume   │  ◄── El disco fisico/logico          │
│  │  (PV)               │      (100Gi, ReadWriteOnce)          │
│  │                     │                                       │
│  └─────────┬───────────┘                                       │
│            │ Bound                                              │
│            │                                                    │
│  ┌─────────▼───────────┐                                       │
│  │ PersistentVolumeClaim│  ◄── "Quiero 50Gi de espacio"       │
│  │  (PVC)              │                                       │
│  │                     │                                       │
│  └─────────┬───────────┘                                       │
│            │ Usado por                                          │
│            │                                                    │
│  ┌─────────▼───────────┐                                       │
│  │        Pod          │                                       │
│  │  ┌───────────────┐  │                                       │
│  │  │  Container    │  │                                       │
│  │  │  /var/data ◄──┼──┼── volumeMount                        │
│  │  └───────────────┘  │                                       │
│  └─────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

## Access Modes

| Modo | Descripcion | Uso Tipico |
|------|-------------|------------|
| `ReadWriteOnce` (RWO) | Un nodo puede leer/escribir | Bases de datos |
| `ReadOnlyMany` (ROX) | Muchos nodos pueden leer | Archivos estaticos |
| `ReadWriteMany` (RWX) | Muchos nodos pueden leer/escribir | Shared storage |

## Reclaim Policy

| Politica | Que pasa al eliminar PVC |
|----------|-------------------------|
| `Retain` | Conserva los datos (manual cleanup) |
| `Delete` | Elimina los datos |

## Provisionamiento Estatico (Manual)

### 1. Crear PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv
spec:
  storageClassName: ""
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/
```

### 2. Crear PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```

### 3. Usar en Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-con-storage
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx
        volumeMounts:                    # Donde montar
        - name: mi-volumen
          mountPath: /var/log/nginx
      volumes:                           # Que montar
      - name: mi-volumen
        persistentVolumeClaim:
          claimName: static-pvc          # Nombre del PVC
```

## Provisionamiento Dinamico (StorageClass)

Con StorageClass, el PV se crea AUTOMATICAMENTE.

### 1. Ver StorageClasses disponibles

```bash
kubectl get sc
```

### 2. Crear PVC (el PV se crea automaticamente)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: auto-pvc
spec:
  storageClassName: standard      # Nombre del StorageClass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

---

# 12. CONFIGMAPS Y SECRETS

## ConfigMaps

Para configuracion **NO sensible**.

### Crear ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mi-config
data:
  APP_COLOR: green
  APP_MODE: production
  DATABASE_URL: postgres://db:5432/mydb
```

### Usar como Variables de Entorno

```yaml
spec:
  containers:
  - name: app
    image: mi-app
    envFrom:
    - configMapRef:
        name: mi-config      # Todas las keys como env vars
```

### Usar como Archivo (Volumen)

```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: mi-config
```

## Secrets

Para datos **SENSIBLES** (passwords, tokens, keys).

**IMPORTANTE:** Los Secrets estan codificados en base64, NO encriptados!

### Crear Secret (CLI)

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=password123
```

### Crear Secret (YAML)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_User: cm9vdA==              # echo -n 'root' | base64
  DB_Password: cGFzc3dvcmQxMjM=  # echo -n 'password123' | base64
```

### Usar Secret

```yaml
spec:
  containers:
  - name: app
    envFrom:
    - secretRef:
        name: db-secret
```

---

# 13. LIFECYCLE - CICLO DE VIDA

## Estrategias de Deployment

Ya vistas en seccion 6: RollingUpdate y Recreate.

## Variables de Entorno

```yaml
spec:
  containers:
  - name: app
    image: mi-app
    env:
    - name: APP_COLOR
      value: pink
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: mi-config
          key: DATABASE_HOST
```

## Health Checks (Probes)

### Tipos de Probes

| Probe | Proposito | Si falla... |
|-------|-----------|-------------|
| **livenessProbe** | "Sigo vivo?" | Reinicia el Pod |
| **readinessProbe** | "Estoy listo para trafico?" | Quita del Service |
| **startupProbe** | "Ya arranque?" | Espera antes de otros probes |

### Metodos de Probe

```yaml
# HTTP Get
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3

# Comando
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5

# TCP Socket
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

### Parametros

| Parametro | Descripcion | Default |
|-----------|-------------|---------|
| `initialDelaySeconds` | Espera antes del primer check | 0 |
| `periodSeconds` | Frecuencia del check | 10 |
| `timeoutSeconds` | Timeout del check | 1 |
| `successThreshold` | Checks exitosos para considerar OK | 1 |
| `failureThreshold` | Checks fallidos para considerar fallo | 3 |

---

# 14. CUOTAS Y LIMITES

## ResourceQuota

Limita el TOTAL de recursos en un Namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "10"                    # Max 10 pods
    requests.cpu: "250m"          # Total CPU requests
    requests.memory: 100Mi        # Total memory requests
    limits.cpu: "500m"            # Total CPU limits
    limits.memory: 500Mi          # Total memory limits
    persistentvolumeclaims: "4"   # Max 4 PVCs
    secrets: "10"                 # Max 10 secrets
```

## LimitRange

Limita recursos POR contenedor.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev
spec:
  limits:
  - type: Container
    default:              # Limit por defecto
      cpu: 200m
      memory: 512Mi
    defaultRequest:       # Request por defecto
      cpu: 100m
      memory: 256Mi
    max:                  # Maximo permitido
      cpu: 500m
      memory: 1024Mi
    min:                  # Minimo permitido
      cpu: 100m
      memory: 100Mi
```

## Requests vs Limits

```yaml
resources:
  requests:              # GARANTIZADOS (scheduler los usa)
    memory: "64Mi"
    cpu: "100m"
  limits:                # MAXIMOS (si los excede: throttle/kill)
    memory: "128Mi"
    cpu: "250m"
```

**Unidades:**
- CPU: `1` = 1 core, `500m` = 0.5 core (500 milicores)
- Memoria: `Mi` = Mebibytes, `Gi` = Gibibytes

---

# 15. TAINTS Y TOLERATIONS

## Que son los Taints?

Un **Taint** en un nodo "rechaza" pods que no lo toleren.

```
┌────────────────────────────────────────────────────────────────┐
│                        NODO                                    │
│                                                                │
│   Taint: tipo=datos:NoSchedule                                │
│                                                                │
│   "No aceptamos Pods que no toleren 'tipo=datos'"            │
│                                                                │
│   Pod normal ────────► RECHAZADO                              │
│                                                                │
│   Pod con toleration ─► ACEPTADO                              │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## Efectos de Taints

| Efecto | Descripcion |
|--------|-------------|
| `NoSchedule` | No programa pods nuevos |
| `PreferNoSchedule` | Evita si es posible |
| `NoExecute` | Expulsa pods existentes |

## Aplicar Taint a un Nodo

```bash
kubectl taint node worker1 tipo=datos:NoSchedule
```

## Crear Pod con Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-especial
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "tipo"
    operator: "Equal"
    value: "datos"
    effect: "NoSchedule"
```

## Eliminar Taint

```bash
kubectl taint node worker1 tipo=datos:NoSchedule-
```

---

# 16. COMANDOS CLI ESENCIALES

## Informacion del Cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <nombre>
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl get pods -n <namespace>
kubectl get pods -A                    # Todos los namespaces
kubectl get pods --show-labels
kubectl get pods -l app=myapp          # Filtrar por label

kubectl describe pod <nombre>
kubectl logs <pod>
kubectl logs <pod> -c <contenedor>
kubectl logs -f <pod>                  # Follow

kubectl exec -it <pod> -- /bin/sh
kubectl exec -it <pod> -c <contenedor> -- /bin/sh

kubectl delete pod <nombre>
kubectl delete pods -l app=myapp
```

## Deployments

```bash
kubectl get deployments
kubectl describe deployment <nombre>
kubectl scale deployment <nombre> --replicas=N

kubectl rollout restart deployment <nombre>
kubectl rollout status deployment <nombre>
kubectl rollout history deployment <nombre>
kubectl rollout undo deployment <nombre>

kubectl set image deployment/<nombre> <contenedor>=<nueva-imagen>
```

## Services

```bash
kubectl get svc
kubectl describe svc <nombre>
kubectl get endpoints
```

## Generar YAML (dry-run)

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --replicas=4 --dry-run=client -o yaml > deploy.yaml
```

## Debugging

```bash
kubectl get events
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods                       # Requiere metrics-server
kubectl top nodes
```

---

# 17. MAPA DE RELACIONES ENTRE RECURSOS

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│                           NAMESPACE                                     │
│                              │                                          │
│     ┌────────────────────────┼────────────────────────┐                │
│     │                        │                        │                │
│     ▼                        ▼                        ▼                │
│ ResourceQuota           LimitRange               ConfigMap/Secret      │
│                                                       │                │
│                                                       │ envFrom/       │
│                                                       │ volumeMount    │
│                                                       ▼                │
│ ┌─────────────────────────────────────────────────────────────────┐   │
│ │                        DEPLOYMENT                                │   │
│ │                             │                                    │   │
│ │                        (crea)                                    │   │
│ │                             ▼                                    │   │
│ │                       REPLICASET                                 │   │
│ │                             │                                    │   │
│ │                        (crea)                                    │   │
│ │                             ▼                                    │   │
│ │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │   │
│ │  │   Pod   │  │   Pod   │  │   Pod   │  │   Pod   │            │   │
│ │  │         │  │         │  │         │  │         │            │   │
│ │  │ labels: │  │ labels: │  │ labels: │  │ labels: │            │   │
│ │  │ app=web │  │ app=web │  │ app=web │  │ app=web │            │   │
│ │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │   │
│ │       │            │            │            │                  │   │
│ └───────┼────────────┼────────────┼────────────┼──────────────────┘   │
│         │            │            │            │                       │
│         └────────────┴─────┬──────┴────────────┘                       │
│                            │                                           │
│                    selector: app=web                                   │
│                            │                                           │
│                            ▼                                           │
│         ┌──────────────────────────────────┐                          │
│         │           SERVICE                │                          │
│         │                                  │                          │
│         │  - ClusterIP (interno)           │                          │
│         │  - NodePort (externo por nodo)   │                          │
│         │  - LoadBalancer (IP publica)     │                          │
│         └────────────────┬─────────────────┘                          │
│                          │                                             │
│                          │ backend                                     │
│                          ▼                                             │
│         ┌──────────────────────────────────┐                          │
│         │           INGRESS                │                          │
│         │                                  │                          │
│         │  host: web.midominio.com        │                          │
│         │  path: /                         │                          │
│         │                                  │                          │
│         └──────────────────────────────────┘                          │
│                          ▲                                             │
│                          │                                             │
│                     INTERNET                                           │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

## Relacion Labels - Selectors

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  DEPLOYMENT                                                          │
│    selector.matchLabels:                                            │
│      app: myapp      ─────────────────────────────┐                 │
│      env: prod                                    │                 │
│                                                   │ DEBEN           │
│  REPLICASET                                       │ COINCIDIR       │
│    template.metadata.labels:                      │                 │
│      app: myapp      ◄────────────────────────────┘                 │
│      env: prod                                                      │
│                                                                      │
│                             │                                        │
│                             ▼                                        │
│  PODs creados con labels:                                           │
│      app: myapp                                                      │
│      env: prod                                                       │
│                                                                      │
│                             ▲                                        │
│                             │                                        │
│  SERVICE                    │ ENCUENTRA                             │
│    selector:                │ POR LABELS                            │
│      app: myapp      ───────┘                                       │
│      env: prod                                                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

# 18. PROYECTO FINAL - ARQUITECTURA DE MICROSERVICIOS

## Arquitectura del Proyecto

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           INTERNET                                       │
│                              │                                           │
│   ┌──────────────────────────┼──────────────────────────┐               │
│   │                          ▼                          │               │
│   │                 INGRESS CONTROLLER                  │               │
│   │                          │                          │               │
│   │    ┌─────────────────────┼─────────────────────┐   │               │
│   │    │                     │                     │   │               │
│   │    ▼                     ▼                     ▼   │               │
│   │ usuario.kubelabs.dev  websocket.usuario...  api.usuario...        │
│   │    │                     │                     │   │               │
│   │    ▼                     ▼                     ▼   │               │
│   │ ┌──────────┐      ┌──────────┐          ┌──────────┐               │
│   │ │  WEBPAGE │      │WEBSOCKET │          │PUBLIC API│               │
│   │ │(frontend)│      │          │          │          │               │
│   │ │ :8080    │      │ :3001    │          │ :3000    │               │
│   │ └──────────┘      └──────────┘          └────┬─────┘               │
│   │                                              │                      │
│   │               NAMESPACE: public              │                      │
│   └──────────────────────────────────────────────┼──────────────────────┘
│                                                  │                      │
│   ┌──────────────────────────────────────────────┼──────────────────────┐
│   │                                              │                      │
│   │               NAMESPACE: private             │                      │
│   │                                              ▼                      │
│   │                                      ┌──────────────┐               │
│   │                                      │ PRIVATE API  │               │
│   │                                      │   :3002      │               │
│   │                                      └──────────────┘               │
│   │                                                                     │
│   └─────────────────────────────────────────────────────────────────────┘
│                                                                          │
│                            CLUSTER K8s                                   │
└──────────────────────────────────────────────────────────────────────────┘
```

## Componentes del Proyecto

### 1. Webpage (Frontend)
- **Namespace:** public
- **Imagen:** `cachac/kubelabs_webapp:1.1.7`
- **Puerto:** 8080
- **Replicas:** 2
- **Ingress:** `usuario.kubelabs.dev`
- **ConfigMap:** Archivo config.js montado

### 2. WebSocket
- **Namespace:** public
- **Imagen:** `cachac/kubelabs_websocket:1.0.6`
- **Puerto:** 3001
- **Replicas:** 1
- **Ingress:** `websocket.usuario.kubelabs.dev/graphql`
- **Secret:** TOKEN_SECRET

### 3. Public API
- **Namespace:** public
- **Imagen:** `cachac/kubelabs_public_api:1.0.0`
- **Puerto:** 3000
- **Replicas:** 2
- **Ingress:** `api.usuario.kubelabs.dev/graphql`
- **ConfigMap:** PRIVATE_API URL

### 4. Private API
- **Namespace:** private
- **Imagen:** `cachac/kubelabs_privateapi:1.0.2`
- **Puerto:** 3002
- **Replicas:** 1
- **NO tiene Ingress** (solo acceso interno)

## Comunicacion entre Namespaces

```yaml
# ConfigMap para Public API
data:
  PRIVATE_API: http://private-api-svc.private.svc.cluster.local:3002/private
```

Formato: `http://<service>.<namespace>.svc.cluster.local:<port>/<path>`

## Recursos por Componente

| Componente | CPU Limit | Memory Limit | CPU Request | Memory Request |
|------------|-----------|--------------|-------------|----------------|
| Webpage | 100m | 100Mi | 10m | 50Mi |
| WebSocket | 250m | 200Mi | 100m | 100Mi |
| Public API | 200m | 200Mi | 100m | 100Mi |
| Private API | 200m | 200Mi | 100m | 100Mi |

## Health Checks

| Componente | Path | Puerto | initialDelay | period |
|------------|------|--------|--------------|--------|
| WebSocket | /healthcheck | 3081 | 10s | 30s |
| Public API | /healthcheck | 3080 | 10s | 30s |
| Private API | /healthcheck | 3082 | 10s | 30s |

## Estrategias de Deployment

- **Webpage:** RollingUpdate con maxUnavailable: 50%
- **WebSocket:** Recreate (elimina todos, crea nuevos)

---

# RESUMEN FINAL

## Flujo de Creacion Tipico

1. **Crear Namespace** para organizar recursos
2. **Crear ConfigMap/Secret** para configuracion
3. **Crear Deployment** con Pods, resources, probes
4. **Crear Service** para exponer internamente
5. **Crear Ingress** para exponer externamente
6. (Opcional) **Crear PVC** para persistencia

## Checklist de Debugging

1. `kubectl get pods` - Estan los Pods Running?
2. `kubectl describe pod <nombre>` - Ver eventos y errores
3. `kubectl logs <pod>` - Ver logs de la aplicacion
4. `kubectl get endpoints` - El Service encontro los Pods?
5. `kubectl describe ingress` - El Ingress tiene backend?
6. `kubectl get events` - Eventos generales del cluster

## Tips para el Examen/Practicas

1. Usa `--dry-run=client -o yaml` para generar YAML rapidamente
2. Verifica que **selector** coincida con **template.labels**
3. Usa `kubectl describe` para ver eventos y errores
4. Los logs estan en `kubectl logs <pod>`
5. Si algo no funciona, revisa los **endpoints** del Service

---

*Documento generado para el curso de Kubernetes - Cenfotec*
*Profesor: Carlos Chacon Calvo*
*Compilado con todos los recursos de los laboratorios del profesor*
