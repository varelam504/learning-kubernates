# 03 - Deployment + Service + Ingress

## Objetivo

Entender como el **Ingress** se conecta con el **Service** para exponer aplicaciones a Internet.

## Diagrama de Relaciones

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                              INTERNET                                       │
│                                  │                                          │
│                                  │ http://webapp.local.test                │
│                                  │                                          │
│                                  ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ INGRESS: webapp-ingress                                                │ │
│  │ ══════════════════════                                                 │ │
│  │                                                                        │ │
│  │   rules:                                                               │ │
│  │     - host: webapp.local.test    ◄── Dominio de acceso               │ │
│  │       http:                                                            │ │
│  │         paths:                                                         │ │
│  │           - path: /               ◄── Ruta                            │ │
│  │             pathType: Prefix                                           │ │
│  │             backend:                                                   │ │
│  │               service:                                                 │ │
│  │                 name: webapp-svc  ◄──┐ Referencia al Service          │ │
│  │                 port:                │                                 │ │
│  │                   number: 80      ◄──┼── Puerto del Service           │ │
│  │                                      │                                 │ │
│  └──────────────────────────────────────┼─────────────────────────────────┘ │
│                                         │                                   │
│                                         │ DEBE COINCIDIR                   │
│                                         │                                   │
│  ┌──────────────────────────────────────▼─────────────────────────────────┐ │
│  │ SERVICE: webapp-svc                                                    │ │
│  │ ═══════════════════                                                    │ │
│  │                                                                        │ │
│  │   metadata:                                                            │ │
│  │     name: webapp-svc         ◄── Este nombre usa el Ingress          │ │
│  │                                                                        │ │
│  │   spec:                                                                │ │
│  │     type: ClusterIP          ◄── Con Ingress, ClusterIP es suficiente│ │
│  │                                                                        │ │
│  │     selector:                                                          │ │
│  │       app: webapp            ◄──┐ Encuentra Pods                      │ │
│  │       tier: frontend            │                                      │ │
│  │                                 │                                      │ │
│  │     ports:                      │                                      │ │
│  │       - port: 80            ◄───┼── Puerto que usa el Ingress         │ │
│  │         targetPort: 8080        │   (backend.service.port.number)     │ │
│  │                                 │                                      │ │
│  └─────────────────────────────────┼──────────────────────────────────────┘ │
│                                    │                                        │
│                                    │ selector DEBE COINCIDIR               │
│                                    │                                        │
│  ┌─────────────────────────────────▼──────────────────────────────────────┐ │
│  │ DEPLOYMENT: webapp-deploy                                              │ │
│  │ ═════════════════════════                                              │ │
│  │                                                                        │ │
│  │   template:                                                            │ │
│  │     metadata:                                                          │ │
│  │       labels:                                                          │ │
│  │         app: webapp          ◄── Service los encuentra por estos     │ │
│  │         tier: frontend           labels                                │ │
│  │                                                                        │ │
│  │     spec:                                                              │ │
│  │       containers:                                                      │ │
│  │         - containerPort: 8080 ◄── Service apunta aqui (targetPort)   │ │
│  │                                                                        │ │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐                           │ │
│  │   │   Pod    │  │   Pod    │  │   Pod    │                           │ │
│  │   │ :8080    │  │ :8080    │  │ :8080    │                           │ │
│  │   └──────────┘  └──────────┘  └──────────┘                           │ │
│  │                                                                        │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Cadena de Conexion Completa

```
                    ┌──────────────────────────────────────────────────────┐
                    │                                                      │
Usuario ──► DNS ──► │ Ingress ──► Service ──► Pod:containerPort           │
                    │                                                      │
                    │ host:      name:       selector:   labels:          │
                    │ webapp.    webapp-svc  app=webapp  app=webapp       │
                    │ local.test                                           │
                    │            port: 80    targetPort: containerPort:   │
                    │                        8080        8080              │
                    │                                                      │
                    └──────────────────────────────────────────────────────┘
```

## Configurar DNS Local

Para probar localmente, edita `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Agrega:
```
127.0.0.1  webapp.local.test
```

## Orden de Aplicacion

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# O todos juntos
kubectl apply -f .
```

## Verificacion

```bash
# Verificar que los Pods estan corriendo
kubectl get pods -n ejemplo-03

# Verificar que el Service tiene endpoints
kubectl get endpoints webapp-svc -n ejemplo-03

# Verificar el Ingress
kubectl get ingress -n ejemplo-03
kubectl describe ingress webapp-ingress -n ejemplo-03

# Probar en el navegador
# http://webapp.local.test
```

## Errores Comunes

### Error 1: Ingress no tiene ADDRESS

```bash
kubectl get ingress -n ejemplo-03
# ADDRESS esta vacio
```

**Causa:** El Ingress Controller no esta instalado.

**Solucion (KIND):**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

### Error 2: Ingress muestra "503 Service Temporarily Unavailable"

**Causas posibles:**
1. El Service no existe o tiene nombre diferente
2. El Service no tiene endpoints (selector no coincide)
3. El puerto en el Ingress no coincide con el puerto del Service

**Verificar:**
```bash
# Existe el Service?
kubectl get svc webapp-svc -n ejemplo-03

# Tiene endpoints?
kubectl get endpoints webapp-svc -n ejemplo-03

# El puerto es correcto?
kubectl describe svc webapp-svc -n ejemplo-03
```

## Siguiente Paso

En el ejemplo 04, agregamos **ConfigMap** y **Secret** para pasar configuracion a los Pods.
