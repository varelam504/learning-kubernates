# 06 - Probes y Resources

## Objetivo

Entender como configurar:
- **Probes**: Health checks (liveness, readiness)
- **Resources**: Requests y limits de CPU/memoria

## Diagrama de Probes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │ POD                                                                    │ │
│  │                                                                        │ │
│  │   ┌────────────────────────────────────────────────────────────────┐  │ │
│  │   │ CONTAINER                                                       │  │ │
│  │   │                                                                 │  │ │
│  │   │   App escuchando en :8080                                      │  │ │
│  │   │                                                                 │  │ │
│  │   │   /healthz ────► "OK" o "Error"                                │  │ │
│  │   │   /ready   ────► "Ready" o "Not Ready"                         │  │ │
│  │   │                                                                 │  │ │
│  │   └────────────────────────────────────────────────────────────────┘  │ │
│  │                                                                        │ │
│  │   KUBERNETES HACE CHECKS:                                             │ │
│  │                                                                        │ │
│  │   ┌────────────────────────────────────────────────────────────────┐  │ │
│  │   │ LIVENESS PROBE: "Sigues vivo?"                                 │  │ │
│  │   │                                                                 │  │ │
│  │   │   httpGet:                                                      │  │ │
│  │   │     path: /healthz                                              │  │ │
│  │   │     port: 8080                                                  │  │ │
│  │   │                                                                 │  │ │
│  │   │   Si FALLA: Kubernetes REINICIA el contenedor                  │  │ │
│  │   │                                                                 │  │ │
│  │   └────────────────────────────────────────────────────────────────┘  │ │
│  │                                                                        │ │
│  │   ┌────────────────────────────────────────────────────────────────┐  │ │
│  │   │ READINESS PROBE: "Estas listo para recibir trafico?"           │  │ │
│  │   │                                                                 │  │ │
│  │   │   httpGet:                                                      │  │ │
│  │   │     path: /ready                                                │  │ │
│  │   │     port: 8080                                                  │  │ │
│  │   │                                                                 │  │ │
│  │   │   Si FALLA: Kubernetes QUITA el Pod del Service (no recibe    │  │ │
│  │   │             trafico, pero NO lo reinicia)                      │  │ │
│  │   │                                                                 │  │ │
│  │   └────────────────────────────────────────────────────────────────┘  │ │
│  │                                                                        │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Diferencia entre Liveness y Readiness

| Probe | Pregunta | Si falla... | Uso tipico |
|-------|----------|-------------|------------|
| **livenessProbe** | "Sigues vivo?" | REINICIA el Pod | Detectar deadlocks |
| **readinessProbe** | "Estas listo?" | QUITA del Service | Startup lento, mantenimiento |

## Diagrama de Resources

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  NODO con 4 CPU y 8Gi de memoria                                           │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐│
│  │                                                                        ││
│  │  REQUESTS: Recursos GARANTIZADOS                                       ││
│  │  ═══════════════════════════════                                       ││
│  │                                                                        ││
│  │  El scheduler usa requests para decidir donde poner el Pod.           ││
│  │  Si pides 500m CPU, el nodo debe tener 500m disponibles.             ││
│  │                                                                        ││
│  │  ┌─────────────────────────────────────────────────────────────────┐  ││
│  │  │ Pod 1                                                            │  ││
│  │  │ requests.cpu: 500m     ◄── "Necesito al menos 500m"            │  ││
│  │  │ requests.memory: 256Mi ◄── "Necesito al menos 256Mi"           │  ││
│  │  └─────────────────────────────────────────────────────────────────┘  ││
│  │                                                                        ││
│  │  LIMITS: Recursos MAXIMOS                                              ││
│  │  ════════════════════════                                              ││
│  │                                                                        ││
│  │  Si el Pod intenta usar mas de lo permitido:                          ││
│  │    - CPU: Se hace THROTTLE (mas lento)                                ││
│  │    - Memoria: Se MATA el contenedor (OOMKilled)                       ││
│  │                                                                        ││
│  │  ┌─────────────────────────────────────────────────────────────────┐  ││
│  │  │ Pod 1                                                            │  ││
│  │  │ limits.cpu: 1000m      ◄── "Maximo 1 CPU"                       │  ││
│  │  │ limits.memory: 512Mi   ◄── "Maximo 512Mi, si no OOMKilled"     │  ││
│  │  └─────────────────────────────────────────────────────────────────┘  ││
│  │                                                                        ││
│  └────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Unidades

### CPU
- `1` = 1 CPU/core completo
- `500m` = 500 milicores = 0.5 CPU
- `250m` = 250 milicores = 0.25 CPU

### Memoria
- `Mi` = Mebibytes (1024 * 1024 bytes)
- `Gi` = Gibibytes (1024 * 1024 * 1024 bytes)
- `256Mi` = 256 Mebibytes
- `1Gi` = 1 Gibibyte

## Parametros de Probes

| Parametro | Descripcion | Default |
|-----------|-------------|---------|
| `initialDelaySeconds` | Espera antes del primer check | 0 |
| `periodSeconds` | Frecuencia del check | 10 |
| `timeoutSeconds` | Timeout de cada check | 1 |
| `successThreshold` | Checks OK para considerar exitoso | 1 |
| `failureThreshold` | Checks fallidos para considerar fallo | 3 |

## Orden de Aplicacion

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Verificacion

```bash
# Ver eventos del Pod (muestra probes)
kubectl describe pod -l app=probes-app -n ejemplo-06

# Ver recursos usados (requiere metrics-server)
kubectl top pods -n ejemplo-06

# Simular fallo de liveness
kubectl exec -it <pod-name> -n ejemplo-06 -- rm /usr/share/nginx/html/index.html
# El Pod se reiniciara automaticamente
```

## Errores Comunes

### Error 1: Pod en CrashLoopBackOff

**Causa:** El livenessProbe falla repetidamente.

**Verificar:**
```bash
kubectl describe pod <pod-name> -n ejemplo-06
# Buscar "Liveness probe failed" en Events
```

### Error 2: OOMKilled

**Causa:** El contenedor uso mas memoria que el limit.

**Solucion:** Aumentar limits.memory.

## Siguiente Paso

En el ejemplo 07, integramos TODO en un proyecto de microservicios.
