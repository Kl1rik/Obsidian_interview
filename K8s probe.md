- Запускаются как curl через kube-probe
### Liveness Probe

- **Liveness Probe** используется для проверки, жив ли контейнер. Если liveness probe не удается, контейнер будет перезапущен.

**Примеры использования:**

- **HTTP liveness probe:**

```yaml
livenessProbe: 
	httpGet: 
		path: /healthz 
		port: 8080 
	initialDelaySeconds: 3 
	periodSeconds: 3
```

- **TCP liveness probe:**
```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3

```
### Readiness Probe
- **Readiness Probe** используется для определения готовности контейнера принимать трафик. Если readiness probe не удается, контейнер будет исключен из балансировщика нагрузки.

**Примеры использования:**

- **HTTP readiness probe:**

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

- **TCP readiness probe:**

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

### Startup Probe

- **Startup Probe** используется для проверки старта контейнера. Если startup probe не удается, Kubernetes будет считать контейнер неуспешно запущенным и перезапустит его. Использование startup probe может помочь предотвратить перезапуски контейнеров, которые имеют длительное время запуска.

**Примеры использования:**

- **HTTP startup probe:**


```yaml
startupProbe:
  httpGet:
    path: /start
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
  failureThreshold: 30
```

- **TCP startup probe:**

```yaml
startupProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
  failureThreshold: 30
```