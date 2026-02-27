# 07 - Proyecto de Microservicios (Practica 2)

## Objetivo

Integrar TODOS los conceptos en un proyecto real de microservicios.

## Arquitectura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              INTERNET                                        │
│                                  │                                           │
│   ┌──────────────────────────────┼──────────────────────────┐               │
│   │                              ▼                          │               │
│   │                     INGRESS CONTROLLER                  │               │
│   │                              │                          │               │
│   │    ┌─────────────────────────┼─────────────────────┐   │               │
│   │    │                         │                     │   │               │
│   │    ▼                         ▼                     ▼   │               │
│   │ usuario.kubelabs.dev   websocket.usuario...   api.usuario...           │
│   │    │                         │                     │   │               │
│   │    ▼                         ▼                     ▼   │               │
│   │ ┌──────────┐          ┌──────────┐          ┌──────────┐               │
│   │ │ WEBPAGE  │          │WEBSOCKET │          │PUBLIC API│               │
│   │ │          │          │          │          │          │               │
│   │ │ :8080    │          │ :3001    │          │ :3000    │               │
│   │ │          │          │          │          │          │               │
│   │ │ Replicas:│          │ Replicas:│          │ Replicas:│               │
│   │ │    2     │          │    1     │          │    2     │               │
│   │ │          │          │          │          │          │               │
│   │ │ ConfigMap│          │  Secret  │          │ ConfigMap│               │
│   │ │ config.js│          │ TOKEN    │          │PRIVATE_API               │
│   │ └──────────┘          └──────────┘          └────┬─────┘               │
│   │                                                  │                      │
│   │                                                  │                      │
│   │               NAMESPACE: public                  │                      │
│   └──────────────────────────────────────────────────┼──────────────────────┘
│                                                      │                      │
│                                                      │ HTTP interno         │
│                                                      │ private-api.private  │
│                                                      │ .svc.cluster.local   │
│                                                      │                      │
│   ┌──────────────────────────────────────────────────▼──────────────────────┐
│   │                                                                         │
│   │               NAMESPACE: private                                        │
│   │                                                                         │
│   │                                      ┌──────────────┐                   │
│   │                                      │ PRIVATE API  │                   │
│   │                                      │              │                   │
│   │                                      │ :3002        │                   │
│   │                                      │              │                   │
│   │                                      │ Replicas: 1  │                   │
│   │                                      │              │                   │
│   │                                      │ NO Ingress   │                   │
│   │                                      │ (solo interno│                   │
│   │                                      └──────────────┘                   │
│   │                                                                         │
│   └─────────────────────────────────────────────────────────────────────────┘
│                                                                              │
│                            CLUSTER K8s                                       │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Mapa de Relaciones

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  NAMESPACE: public                                                          │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐│
│  │                                                                        ││
│  │  ConfigMap: webpage-config         Secret: websocket-secret           ││
│  │  ═══════════════════════          ════════════════════════            ││
│  │    name: webpage-config             name: websocket-secret            ││
│  │          │                                │                            ││
│  │          │ volumes.configMap.name        │ envFrom.secretRef.name     ││
│  │          │                                │                            ││
│  │          ▼                                ▼                            ││
│  │  Deployment: webpage-deploy       Deployment: websocket-deploy        ││
│  │  ══════════════════════════       ═══════════════════════════         ││
│  │    selector.matchLabels:            selector.matchLabels:             ││
│  │      app: webpage      ◄────┐         app: websocket    ◄────┐       ││
│  │      tier: frontend         │         tier: backend          │       ││
│  │                             │                                │       ││
│  │    template.labels:         │       template.labels:         │       ││
│  │      app: webpage      ─────┘         app: websocket    ─────┘       ││
│  │      tier: frontend   ─────────┐      tier: backend    ─────────┐    ││
│  │                                │                                │    ││
│  │    containerPort: 8080   ◄──┐  │    containerPort: 3001   ◄──┐ │    ││
│  │                             │  │                             │ │    ││
│  │                             │  │                             │ │    ││
│  │  Service: webpage-svc       │  │  Service: websocket-svc     │ │    ││
│  │  ════════════════════       │  │  ═══════════════════════    │ │    ││
│  │    name: webpage-svc   ◄────┼──┼─┐  name: websocket-svc ◄────┼─┼─┐  ││
│  │                             │  │ │                           │ │ │  ││
│  │    selector:                │  │ │  selector:                │ │ │  ││
│  │      app: webpage      ─────┘  │ │    app: websocket    ─────┘ │ │  ││
│  │      tier: frontend   ─────────┘ │    tier: backend    ────────┘ │  ││
│  │                                  │                               │  ││
│  │    port: 80                      │  port: 80                     │  ││
│  │    targetPort: 8080   ───────────┘  targetPort: 3001  ───────────┘  ││
│  │                                                                      ││
│  │                                                                      ││
│  │  ConfigMap: publicapi-config                                         ││
│  │  ═══════════════════════════                                         ││
│  │    name: publicapi-config                                            ││
│  │    data:                                                             ││
│  │      PRIVATE_API: http://private-api.private.svc.cluster.local:3002 ││
│  │          │                                           │               ││
│  │          │ envFrom.configMapRef.name                │               ││
│  │          │                                           │               ││
│  │          ▼                                           │               ││
│  │  Deployment: publicapi-deploy                        │               ││
│  │  ════════════════════════════                        │               ││
│  │    selector.matchLabels:                             │               ││
│  │      app: public-api   ◄────┐                       │               ││
│  │      tier: backend          │                       │               ││
│  │                             │                       │               ││
│  │    template.labels:         │                       │               ││
│  │      app: public-api   ─────┘                       │               ││
│  │      tier: backend    ──────────┐                   │               ││
│  │                                 │                   │               ││
│  │    containerPort: 3000    ◄──┐  │                   │               ││
│  │                              │  │                   │               ││
│  │  Service: publicapi-svc      │  │                   │               ││
│  │  ══════════════════════      │  │                   │               ││
│  │    name: publicapi-svc  ◄────┼──┼──┐                │               ││
│  │                              │  │  │                │               ││
│  │    selector:                 │  │  │                │               ││
│  │      app: public-api    ─────┘  │  │                │               ││
│  │      tier: backend     ─────────┘  │                │               ││
│  │                                    │                │               ││
│  │    port: 80                        │                │               ││
│  │    targetPort: 3000   ─────────────┘                │               ││
│  │                                                      │               ││
│  └──────────────────────────────────────────────────────┼───────────────┘│
│                                                         │                │
│                                                         │ DNS INTERNO    │
│                                                         │                │
│  NAMESPACE: private                                     ▼                │
│  ┌──────────────────────────────────────────────────────────────────────┐│
│  │                                                                      ││
│  │  Deployment: privateapi-deploy                                       ││
│  │  ═════════════════════════════                                       ││
│  │    selector.matchLabels:                                             ││
│  │      app: private-api   ◄────┐                                      ││
│  │      tier: backend           │                                      ││
│  │                              │                                      ││
│  │    template.labels:          │                                      ││
│  │      app: private-api   ─────┘                                      ││
│  │      tier: backend     ──────────┐                                  ││
│  │                                  │                                  ││
│  │    containerPort: 3002     ◄──┐  │                                  ││
│  │                               │  │                                  ││
│  │  Service: private-api         │  │                                  ││
│  │  ════════════════════         │  │                                  ││
│  │    name: private-api     ◄────┼──┼──── ConfigMap.PRIVATE_API lo usa ││
│  │                               │  │                                  ││
│  │    selector:                  │  │                                  ││
│  │      app: private-api    ─────┘  │                                  ││
│  │      tier: backend      ─────────┘                                  ││
│  │                                                                      ││
│  │    port: 3002                                                        ││
│  │    targetPort: 3002                                                  ││
│  │                                                                      ││
│  └──────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│  INGRESS: microservices-ingress                                         │
│  ═══════════════════════════════                                        │
│    rules:                                                                │
│      - host: usuario.kubelabs.dev                                       │
│          backend.service.name: webpage-svc      ◄── Service.name        │
│          backend.service.port.number: 80        ◄── Service.port        │
│                                                                          │
│      - host: websocket.usuario.kubelabs.dev                             │
│          backend.service.name: websocket-svc    ◄── Service.name        │
│          backend.service.port.number: 80        ◄── Service.port        │
│                                                                          │
│      - host: api.usuario.kubelabs.dev                                   │
│          backend.service.name: publicapi-svc    ◄── Service.name        │
│          backend.service.port.number: 80        ◄── Service.port        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Archivos del Proyecto

| # | Archivo | Descripcion |
|---|---------|-------------|
| 00 | namespaces.yaml | Crea namespaces public y private |
| 01 | configmaps.yaml | ConfigMaps para webpage y public-api |
| 02 | secrets.yaml | Secret para websocket |
| 03 | webpage.yaml | Deployment + Service del frontend |
| 04 | websocket.yaml | Deployment + Service del websocket |
| 05 | public-api.yaml | Deployment + Service del public API |
| 06 | private-api.yaml | Deployment + Service del private API |
| 07 | ingress.yaml | Ingress para exponer los servicios |

## Orden de Aplicacion

```bash
# 1. Namespaces primero
kubectl apply -f 00-namespaces.yaml

# 2. ConfigMaps y Secrets (antes de los Deployments)
kubectl apply -f 01-configmaps.yaml
kubectl apply -f 02-secrets.yaml

# 3. Deployments y Services
kubectl apply -f 03-webpage.yaml
kubectl apply -f 04-websocket.yaml
kubectl apply -f 05-public-api.yaml
kubectl apply -f 06-private-api.yaml

# 4. Ingress al final
kubectl apply -f 07-ingress.yaml

# O todo de una vez (respeta el orden numerico)
kubectl apply -f .
```

## Verificacion Completa

```bash
# Ver namespaces
kubectl get ns

# Ver recursos en public
kubectl get all -n public
kubectl get configmap,secret -n public
kubectl get endpoints -n public

# Ver recursos en private
kubectl get all -n private
kubectl get endpoints -n private

# Ver Ingress
kubectl get ingress -n public
kubectl describe ingress microservices-ingress -n public
```

## DNS Interno

Para que public-api se comunique con private-api:

```
http://<service-name>.<namespace>.svc.cluster.local:<port>
http://private-api.private.svc.cluster.local:3002
```

## Probar Localmente

Agregar a `/etc/hosts`:
```
127.0.0.1  usuario.kubelabs.dev
127.0.0.1  websocket.usuario.kubelabs.dev
127.0.0.1  api.usuario.kubelabs.dev
```

Luego acceder:
- http://usuario.kubelabs.dev
- http://api.usuario.kubelabs.dev/graphql
