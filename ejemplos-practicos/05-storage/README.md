# 05 - Storage (PersistentVolumeClaim)

## Objetivo

Entender como usar almacenamiento persistente con:
- **PersistentVolumeClaim (PVC)**: Solicita almacenamiento
- **volumeMounts**: Donde se monta en el contenedor

## Diagrama de Relaciones

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  STORAGECLASS: standard (default en KIND)                                  │
│  ══════════════════════════════════════════                                │
│    provisioner: rancher.io/local-path                                      │
│    (Crea PV automaticamente cuando se pide PVC)                           │
│           │                                                                 │
│           │ Provisiona automaticamente                                     │
│           ▼                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ PERSISTENTVOLUME (creado automaticamente)                            │  │
│  │   capacity: 100Mi                                                     │  │
│  │   accessModes: ReadWriteOnce                                          │  │
│  │   status: Bound                                                       │  │
│  └────────────────────────────────────┬─────────────────────────────────┘  │
│                                       │                                    │
│                                       │ Bound                              │
│                                       │                                    │
│  ┌────────────────────────────────────▼─────────────────────────────────┐  │
│  │ PERSISTENTVOLUMECLAIM: app-pvc                                       │  │
│  │ ══════════════════════════════════                                   │  │
│  │   spec:                                                               │  │
│  │     storageClassName: standard                                        │  │
│  │     accessModes:                                                      │  │
│  │       - ReadWriteOnce                                                 │  │
│  │     resources:                                                        │  │
│  │       requests:                                                       │  │
│  │         storage: 100Mi                                                │  │
│  │                                                                       │  │
│  │   status: Bound                                                       │  │
│  └────────────────────────────────────┬─────────────────────────────────┘  │
│                                       │                                    │
│                                       │ Usado por                         │
│                                       │                                    │
│  ┌────────────────────────────────────▼─────────────────────────────────┐  │
│  │ DEPLOYMENT                                                           │  │
│  │                                                                       │  │
│  │   spec:                                                               │  │
│  │     containers:                                                       │  │
│  │       - name: app                                                     │  │
│  │         volumeMounts:                                                 │  │
│  │           - name: data-volume   ◄──┐                                 │  │
│  │             mountPath: /data       │ MISMO NOMBRE                    │  │
│  │                                    │                                  │  │
│  │     volumes:                       │                                  │  │
│  │       - name: data-volume      ────┘                                  │  │
│  │         persistentVolumeClaim:                                        │  │
│  │           claimName: app-pvc   ◄── PVC.metadata.name                 │  │
│  │                                                                       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  DENTRO DEL CONTENEDOR:                                                    │
│  ══════════════════════                                                    │
│                                                                             │
│  $ df -h /data                                                             │
│  Filesystem      Size  Used Avail Use% Mounted on                         │
│  /dev/sda1       100Mi 0    100Mi  0%  /data                              │
│                                                                             │
│  Los datos en /data PERSISTEN aunque el Pod se reinicie                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Access Modes

| Modo | Sigla | Descripcion |
|------|-------|-------------|
| ReadWriteOnce | RWO | Un solo nodo puede leer/escribir |
| ReadOnlyMany | ROX | Multiples nodos pueden leer |
| ReadWriteMany | RWX | Multiples nodos pueden leer/escribir |

## Orden de Aplicacion

```bash
# IMPORTANTE: PVC DEBE existir ANTES del Deployment
kubectl apply -f namespace.yaml
kubectl apply -f pvc.yaml            # Primero
kubectl apply -f deployment.yaml     # Despues (usa el PVC)
kubectl apply -f service.yaml
```

## Verificacion

```bash
# Ver PVC
kubectl get pvc -n ejemplo-05
# STATUS debe ser "Bound"

# Ver PV (creado automaticamente)
kubectl get pv

# Ver que el Pod tiene el volumen montado
kubectl describe pod -l app=storage-app -n ejemplo-05
# Buscar "Volumes" y "Mounts"

# Probar persistencia
# 1. Crear un archivo
kubectl exec -it <pod-name> -n ejemplo-05 -- sh -c "echo 'Hola' > /data/test.txt"

# 2. Eliminar el Pod
kubectl delete pod <pod-name> -n ejemplo-05

# 3. El Deployment crea un nuevo Pod

# 4. Verificar que el archivo persiste
kubectl exec -it <nuevo-pod-name> -n ejemplo-05 -- cat /data/test.txt
# Debe mostrar "Hola"
```

## Errores Comunes

### Error 1: PVC en estado Pending

```bash
kubectl get pvc -n ejemplo-05
# STATUS: Pending
```

**Causas:**
1. No hay StorageClass disponible
2. El StorageClass no existe

**Verificar:**
```bash
kubectl get storageclass
kubectl describe pvc app-pvc -n ejemplo-05
```

### Error 2: Pod en estado Pending esperando PVC

```bash
kubectl get pods -n ejemplo-05
# STATUS: Pending
```

**Causa:** El PVC no esta Bound.

**Verificar:**
```bash
kubectl describe pod <pod-name> -n ejemplo-05
# Buscar en Events
```

## Siguiente Paso

En el ejemplo 06, agregamos **Probes** y **Resources** para health checks y limites.
