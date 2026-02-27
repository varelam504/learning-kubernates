# 01 - Pod Basico

## Objetivo

Entender la estructura basica de un Pod y la importancia de los **labels**.

## Diagrama

```
┌──────────────────────────────────────────┐
│               POD: nginx-pod             │
│                                          │
│   Labels:                                │
│     app: nginx       ◄── Identificador   │
│     env: desarrollo  ◄── Ambiente        │
│                                          │
│   ┌────────────────────────────────┐    │
│   │     CONTAINER: nginx           │    │
│   │     Image: nginx:1.21          │    │
│   │     Port: 80                   │    │
│   └────────────────────────────────┘    │
│                                          │
│   IP: 10.244.x.x (asignada por K8s)     │
└──────────────────────────────────────────┘
```

## Por que son importantes los Labels?

Los **labels** son pares clave-valor que identifican objetos. Son FUNDAMENTALES porque:

1. **Services** encuentran Pods por labels
2. **ReplicaSets/Deployments** gestionan Pods por labels
3. Puedes filtrar recursos con `kubectl get pods -l app=nginx`

```yaml
labels:
  app: nginx          # Nombre de la aplicacion
  env: desarrollo     # Ambiente (dev, staging, prod)
  version: "1.0"      # Version
  team: backend       # Equipo responsable
```

## Archivo: pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    env: desarrollo
spec:
  containers:
    - name: nginx
      image: nginx:1.21
      ports:
        - containerPort: 80
```

## Comandos

```bash
# Aplicar
kubectl apply -f pod.yaml

# Verificar
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels

# Filtrar por label
kubectl get pods -l app=nginx
kubectl get pods -l env=desarrollo

# Ver detalles
kubectl describe pod nginx-pod

# Ver logs
kubectl logs nginx-pod

# Ejecutar comando dentro del Pod
kubectl exec -it nginx-pod -- /bin/sh

# Eliminar
kubectl delete -f pod.yaml
```

## Siguiente Paso

En el ejemplo 02, vamos a usar un **Deployment** que gestiona multiples Pods usando **matchLabels**, y un **Service** que los encuentra usando **selector**.
