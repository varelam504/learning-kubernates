# EJEMPLOS PRACTICOS DE KUBERNETES

## Estructura de Ejemplos

Cada ejemplo está diseñado para construir sobre el anterior. Los conceptos clave (labels, selectors, referencias) están **correctamente amarrados** en cada nivel.

```
ejemplos-practicos/
│
├── 01-pod-basico/           # Pod simple con labels
│   └── README.md
│   └── pod.yaml
│
├── 02-deployment-service/   # Deployment + Service (matchLabels)
│   └── README.md
│   └── namespace.yaml
│   └── deployment.yaml
│   └── service.yaml
│
├── 03-deployment-ingress/   # Agregamos Ingress
│   └── README.md
│   └── namespace.yaml
│   └── deployment.yaml
│   └── service.yaml
│   └── ingress.yaml
│
├── 04-configmap-secret/     # ConfigMap y Secret como env/volume
│   └── README.md
│   └── namespace.yaml
│   └── configmap.yaml
│   └── secret.yaml
│   └── deployment.yaml
│   └── service.yaml
│
├── 05-storage/              # PVC y volumeMounts
│   └── README.md
│   └── namespace.yaml
│   └── pvc.yaml
│   └── deployment.yaml
│   └── service.yaml
│
├── 06-probes-resources/     # Health checks y limits
│   └── README.md
│   └── namespace.yaml
│   └── deployment.yaml
│   └── service.yaml
│
└── 07-proyecto-microservicios/  # Proyecto completo
    └── README.md
    └── 00-namespaces.yaml
    └── 01-configmaps.yaml
    └── 02-secrets.yaml
    └── 03-webpage.yaml
    └── 04-websocket.yaml
    └── 05-public-api.yaml
    └── 06-private-api.yaml
    └── 07-ingress.yaml
```

---

## Diagrama de Relaciones Clave

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ DEPLOYMENT                                                           │   │
│  │                                                                      │   │
│  │   metadata:                                                          │   │
│  │     name: mi-app          ◄──────────────────────────────────────┐  │   │
│  │                                                                   │  │   │
│  │   spec:                                                           │  │   │
│  │     selector:                                                     │  │   │
│  │       matchLabels:        ◄───┐                                  │  │   │
│  │         app: mi-app           │ DEBEN SER                        │  │   │
│  │         tier: frontend        │ IDENTICOS                        │  │   │
│  │                               │                                   │  │   │
│  │     template:                 │                                   │  │   │
│  │       metadata:               │                                   │  │   │
│  │         labels:           ────┘                                   │  │   │
│  │           app: mi-app                                             │  │   │
│  │           tier: frontend                                          │  │   │
│  │                                                                   │  │   │
│  │       spec:                                                       │  │   │
│  │         containers:                                               │  │   │
│  │           - name: web                                             │  │   │
│  │             envFrom:                                              │  │   │
│  │               - configMapRef:                                     │  │   │
│  │                   name: mi-config  ◄──── Referencia a ConfigMap  │  │   │
│  │               - secretRef:                                        │  │   │
│  │                   name: mi-secret  ◄──── Referencia a Secret     │  │   │
│  │             volumeMounts:                                         │  │   │
│  │               - name: data-vol                                    │  │   │
│  │                 mountPath: /data   ◄──┐                          │  │   │
│  │         volumes:                      │ MISMO NOMBRE             │  │   │
│  │           - name: data-vol        ────┘                          │  │   │
│  │             persistentVolumeClaim:                                │  │   │
│  │               claimName: mi-pvc    ◄──── Referencia a PVC        │  │   │
│  │                                                                   │  │   │
│  └───────────────────────────────────────────────────────────────────┘  │   │
│                                                                          │   │
│  ┌─────────────────────────────────────────────────────────────────────┐│   │
│  │ SERVICE                                                              ││   │
│  │                                                                      ││   │
│  │   metadata:                                                          ││   │
│  │     name: mi-app-svc      ◄───────────────────────────────────────┐ ││   │
│  │                                                                    │ ││   │
│  │   spec:                                                            │ ││   │
│  │     selector:             ◄───┐                                   │ ││   │
│  │       app: mi-app             │ DEBEN COINCIDIR CON              │ ││   │
│  │       tier: frontend          │ template.metadata.labels          │ ││   │
│  │                           ────┘ (para encontrar los Pods)         │ ││   │
│  │     ports:                                                         │ ││   │
│  │       - port: 80              ◄──── Puerto del Service           │ ││   │
│  │         targetPort: 8080      ◄──── Puerto del Contenedor        │ ││   │
│  │                                                                    │ ││   │
│  └────────────────────────────────────────────────────────────────────┘││   │
│                                                                         ││   │
│  ┌─────────────────────────────────────────────────────────────────────┐│   │
│  │ INGRESS                                                              │   │
│  │                                                                      │   │
│  │   spec:                                                              │   │
│  │     rules:                                                           │   │
│  │       - host: mi-app.ejemplo.com                                    │   │
│  │         http:                                                        │   │
│  │           paths:                                                     │   │
│  │             - backend:                                               │   │
│  │                 service:                                             │   │
│  │                   name: mi-app-svc    ◄── Referencia al Service    │   │
│  │                   port:                                              │   │
│  │                     number: 80        ◄── Puerto del Service        │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Como Usar estos Ejemplos

### Opcion 1: Aplicar un ejemplo completo

```bash
# Ir a la carpeta del ejemplo
cd 02-deployment-service

# Aplicar todos los archivos en orden
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# O aplicar todos de una vez
kubectl apply -f .
```

### Opcion 2: Aplicar de forma recursiva

```bash
# Aplicar todos los ejemplos de una carpeta
kubectl apply -R -f 07-proyecto-microservicios/
```

### Limpiar

```bash
# Eliminar recursos de un ejemplo
kubectl delete -f .

# O eliminar el namespace completo
kubectl delete namespace mi-namespace
```

---

## Validaciones Importantes

Antes de aplicar, verifica:

1. **matchLabels = template.labels**
   ```bash
   # Si no coinciden, el Deployment no puede gestionar los Pods
   ```

2. **Service.selector = Pod.labels**
   ```bash
   # Verificar que el Service encuentra los Pods
   kubectl get endpoints mi-servicio
   ```

3. **Ingress.backend.service.name = Service.metadata.name**
   ```bash
   # Verificar que el Ingress apunta al Service correcto
   kubectl describe ingress mi-ingress
   ```

4. **Referencias de ConfigMap/Secret existen**
   ```bash
   kubectl get configmap mi-config
   kubectl get secret mi-secret
   ```

5. **PVC existe antes de crear el Deployment**
   ```bash
   kubectl get pvc mi-pvc
   ```

---

## Orden de Aplicacion Recomendado

```
1. Namespace          # Primero el "contenedor" logico
2. ConfigMap/Secret   # Configuracion que van a usar los Pods
3. PVC                # Almacenamiento (si aplica)
4. Deployment         # Los Pods (usan ConfigMap, Secret, PVC)
5. Service            # Expone los Pods internamente
6. Ingress            # Expone el Service externamente
```

---

## Indice de Ejemplos

| # | Ejemplo | Conceptos | Dificultad |
|---|---------|-----------|------------|
| 01 | [Pod Basico](./01-pod-basico/) | Pod, labels, contenedor | Facil |
| 02 | [Deployment + Service](./02-deployment-service/) | Deployment, ReplicaSet, Service, matchLabels, selector | Facil |
| 03 | [Deployment + Ingress](./03-deployment-ingress/) | Ingress, hosts, paths, backend | Medio |
| 04 | [ConfigMap + Secret](./04-configmap-secret/) | ConfigMap, Secret, envFrom, volumeMount | Medio |
| 05 | [Storage](./05-storage/) | PVC, volumeMount, persistencia | Medio |
| 06 | [Probes + Resources](./06-probes-resources/) | liveness, readiness, requests, limits | Medio |
| 07 | [Proyecto Microservicios](./07-proyecto-microservicios/) | Todo integrado | Avanzado |

---

*Cada carpeta tiene su propio README con explicaciones detalladas.*
