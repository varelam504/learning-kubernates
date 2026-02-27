# 02 - Deployment + Service

## Objetivo

Entender la relacion entre:
- **Deployment.selector.matchLabels** y **template.metadata.labels**
- **Service.selector** y **Pod.labels**

## Diagrama de Relaciones

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  DEPLOYMENT: web-deploy                                                     │
│  ═══════════════════════                                                    │
│                                                                             │
│    selector:                                                                │
│      matchLabels:           ◄────────┐                                     │
│        app: web-app                  │                                     │
│        tier: frontend                │ DEBEN SER                           │
│                                      │ IDENTICOS                           │
│    template:                         │                                     │
│      metadata:                       │                                     │
│        labels:              ─────────┘                                     │
│          app: web-app       ─────────────────────────┐                     │
│          tier: frontend                              │                     │
│                                                      │                     │
│      spec:                                           │                     │
│        containers:                                   │                     │
│          - name: web                                 │                     │
│            image: nginx                              │                     │
│            ports:                                    │                     │
│              - containerPort: 80  ◄──────────────────┼──┐                  │
│                                                      │  │                  │
│                                                      │  │                  │
│  Crea estos PODS:                                    │  │                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │  │                  │
│  │   Pod    │  │   Pod    │  │   Pod    │          │  │                  │
│  │ app:     │  │ app:     │  │ app:     │          │  │                  │
│  │ web-app  │  │ web-app  │  │ web-app  │  ◄───────┘  │                  │
│  │ tier:    │  │ tier:    │  │ tier:    │             │                  │
│  │ frontend │  │ frontend │  │ frontend │             │                  │
│  │          │  │          │  │          │             │                  │
│  │ :80      │  │ :80      │  │ :80      │             │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘             │                  │
│       │             │             │                    │                  │
│       └─────────────┼─────────────┘                    │                  │
│                     │                                  │                  │
│                     │ selector: app=web-app            │                  │
│                     │          tier=frontend           │                  │
│                     │                                  │                  │
│  SERVICE: web-svc   ▼                                  │                  │
│  ════════════════════                                  │                  │
│    selector:        ◄── DEBE COINCIDIR con Pod.labels │                  │
│      app: web-app                                      │                  │
│      tier: frontend                                    │                  │
│                                                        │                  │
│    ports:                                              │                  │
│      - port: 80           ◄── Puerto del Service      │                  │
│        targetPort: 80     ◄── Puerto del Container ───┘                  │
│                                                                           │
│    ENDPOINTS: 10.244.1.5:80, 10.244.2.3:80, 10.244.3.7:80               │
│               (IPs de los Pods encontrados)                              │
│                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Archivos

### 1. namespace.yaml
Primero creamos el namespace para organizar los recursos.

### 2. deployment.yaml
Crea el Deployment con la configuracion correcta de labels.

### 3. service.yaml
Crea el Service que encuentra los Pods por sus labels.

## Orden de Aplicacion

```bash
kubectl apply -f namespace.yaml     # 1. Primero el namespace
kubectl apply -f deployment.yaml    # 2. Luego el deployment
kubectl apply -f service.yaml       # 3. Finalmente el service

# O todos juntos (K8s respeta dependencias)
kubectl apply -f .
```

## Comandos de Verificacion

```bash
# Ver los Pods creados
kubectl get pods -n ejemplo-02

# Ver las labels de los Pods
kubectl get pods -n ejemplo-02 --show-labels

# Ver el Deployment
kubectl get deployment -n ejemplo-02

# Ver el Service
kubectl get svc -n ejemplo-02

# VER ENDPOINTS - Muy importante!
# Si esta vacio, el selector no coincide con los Pod labels
kubectl get endpoints web-svc -n ejemplo-02

# Describir el Service para ver los endpoints
kubectl describe svc web-svc -n ejemplo-02
```

## Errores Comunes

### Error 1: matchLabels no coincide con template.labels

```yaml
# MALO - No coinciden!
selector:
  matchLabels:
    app: web-app
template:
  metadata:
    labels:
      app: webapp    # <-- Diferente! Falta el guion
```

**Mensaje de error:**
```
`selector` does not match template `labels`
```

### Error 2: Service no encuentra Pods

```yaml
# Service
selector:
  app: web          # <-- Solo busca "app: web"

# Pero los Pods tienen:
labels:
  app: web-app      # <-- Es "app: web-app", no "app: web"
```

**Como detectarlo:**
```bash
kubectl get endpoints web-svc -n ejemplo-02
# Si muestra <none>, el selector no encontro ningun Pod
```

## Siguiente Paso

En el ejemplo 03, agregamos un **Ingress** que expone el Service a Internet usando un dominio.
