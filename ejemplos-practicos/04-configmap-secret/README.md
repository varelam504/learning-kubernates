# 04 - ConfigMap y Secret

## Objetivo

Entender como pasar configuracion a los Pods usando:
- **ConfigMap**: Configuracion NO sensible
- **Secret**: Datos SENSIBLES (passwords, tokens)

## Diagrama de Relaciones

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  CONFIGMAP: app-config                    SECRET: app-secret               │
│  ═════════════════════                    ══════════════════               │
│    data:                                    data:                          │
│      APP_COLOR: blue                          DB_PASSWORD: (base64)        │
│      APP_MODE: production                     API_KEY: (base64)            │
│      DATABASE_HOST: db.example.com                                         │
│           │                                        │                       │
│           │                                        │                       │
│           │ envFrom:                               │ envFrom:              │
│           │   configMapRef:                        │   secretRef:          │
│           │     name: app-config                   │     name: app-secret  │
│           │                                        │                       │
│           └────────────────────┬───────────────────┘                       │
│                                │                                           │
│                                ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │ DEPLOYMENT                                                           │  │
│  │                                                                      │  │
│  │   spec:                                                              │  │
│  │     containers:                                                      │  │
│  │       - name: app                                                    │  │
│  │         envFrom:                                                     │  │
│  │           - configMapRef:                                            │  │
│  │               name: app-config    ◄── Todas las keys como env vars  │  │
│  │           - secretRef:                                               │  │
│  │               name: app-secret    ◄── Todas las keys como env vars  │  │
│  │                                                                      │  │
│  │         volumeMounts:                                                │  │
│  │           - name: config-vol      ◄──┐                              │  │
│  │             mountPath: /etc/config    │ MISMO NOMBRE                │  │
│  │                                       │                              │  │
│  │     volumes:                          │                              │  │
│  │       - name: config-vol          ────┘                              │  │
│  │         configMap:                                                   │  │
│  │           name: app-config        ◄── Monta ConfigMap como archivos │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  DENTRO DEL CONTENEDOR:                                                    │
│  ══════════════════════                                                    │
│                                                                             │
│  Variables de entorno:                                                     │
│    $ echo $APP_COLOR                                                       │
│    blue                                                                     │
│    $ echo $DB_PASSWORD                                                     │
│    mi-password-secreto                                                     │
│                                                                             │
│  Archivos (si se monta como volumen):                                     │
│    $ cat /etc/config/APP_COLOR                                             │
│    blue                                                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Dos Formas de Usar ConfigMap/Secret

### 1. Como Variables de Entorno (envFrom)

```yaml
envFrom:
  - configMapRef:
      name: app-config       # Todas las keys se convierten en env vars
  - secretRef:
      name: app-secret       # Todas las keys se convierten en env vars
```

### 2. Como Archivos (Volume)

```yaml
volumeMounts:
  - name: config-vol
    mountPath: /etc/config   # Directorio donde se montan los archivos

volumes:
  - name: config-vol
    configMap:
      name: app-config       # Cada key se convierte en un archivo
```

## Orden de Aplicacion

```bash
# IMPORTANTE: ConfigMap y Secret DEBEN existir ANTES del Deployment
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml    # Primero
kubectl apply -f secret.yaml       # Primero
kubectl apply -f deployment.yaml   # Despues (usa los anteriores)
kubectl apply -f service.yaml
```

## Verificacion

```bash
# Ver ConfigMap
kubectl get configmap app-config -n ejemplo-04
kubectl describe configmap app-config -n ejemplo-04

# Ver Secret (no muestra valores!)
kubectl get secret app-secret -n ejemplo-04
kubectl describe secret app-secret -n ejemplo-04

# Ver valores del Secret (decodificados)
kubectl get secret app-secret -n ejemplo-04 -o jsonpath='{.data.DB_PASSWORD}' | base64 -d

# Verificar que el Pod tiene las variables
kubectl exec -it <pod-name> -n ejemplo-04 -- env | grep APP
kubectl exec -it <pod-name> -n ejemplo-04 -- env | grep DB
```

## Errores Comunes

### Error 1: ConfigMap/Secret no existe

```
Error: configmaps "app-config" not found
```

**Causa:** El Deployment intenta usar un ConfigMap que no existe.

**Solucion:** Aplicar ConfigMap antes del Deployment.

### Error 2: Pod en estado CreateContainerConfigError

```bash
kubectl get pods
# STATUS: CreateContainerConfigError
```

**Causa:** El ConfigMap o Secret referenciado no existe.

**Verificar:**
```bash
kubectl describe pod <pod-name> -n ejemplo-04
# Buscar en Events el error especifico
```

## Siguiente Paso

En el ejemplo 05, agregamos **PersistentVolumeClaim** para almacenamiento persistente.
