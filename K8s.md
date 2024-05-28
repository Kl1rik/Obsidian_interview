![](https://k8sdesiredstate.github.io/k8s_logo.png)

---
## **Компоненты кластера**
![[components-of-k8s.png]]
- Control plane
	- Компоненты
		- Kube-scheduler
		- Api-gateway
		- Controller
			- manager
			- Cloud-manager
		- Etcd
- Kubelet
- Dockerd(k3s)
- ==Максимальное количество подов на узле : 110==
## **Масштабирование внутри кластера**
- HPA 
- VPA 
- PBD 
## **Ресурсы кластера**
---
### **Deployment**
- Strategy
	- Recreate(сначала убивает старые поды, потом создает новые)
	- Rolling update(последовательно убивает поды по 1..110/1..100% и создает новые)
	- Хорошо подходит для приложений без необходимости поддерживать свое состояние между сессиями 
### StatefulSet
- Поддерживает PVC
- В отличие от Deployment не изменяется динамически
- Необходим для приложений ,для которых нужно сохранять состояние 
---
### **PV**
- Пример yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
```
- Описывает потстоянные тома для хранения данных
- Поддерживает разные storage class
- Подключение в Deployment/StatefulClass через persistentVolumeClaim
### **PVC**
- Пример yaml для привязки конкретного PVC к конкретному PV
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 10Gi

```
- Является абстракцией для управления **PV** и  **StorageClass**
- Позволяет создавать  **PV** без управления руками,а при наличии  **PV** с существующим именем подключается к нему
- Поддерживает настройку **accessModes** (Политик чтения и запписи) и **requests**
-  **accessModes**
	- ReadWriteOnce
	- ReadOnlyMany
	- ReadWriteMany
- **ReclaimPolicy**
	- Нужен для задания сценария после удаления **PVC**
	- ==Delete==
	- ==Retain (Сохранить)**==
	- ==Recycle (Переработать)==
		- Форматируется PV и отправляется в общий пул со статусо Released
### **PV Provisioner**
- Программа ,которая удет создавать тома нужного размера с нужнфми политиками через описание в **StorageClass**
- При задании через CLI/APi и создает том нужного размера,а потом создает нужный yaml манифест в кластере 
### **StorageClass**
- Пример yaml
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-vol-default
provisioner: vendor-name.example/magicstorage
parameters:
  resturl: "http://192.168.10.100:8080"
  restuser: ""
  secretNamespace: ""
  secretName: ""
allowVolumeExpansion: true

```
- Используется для определения классов хранения, которые предоставляют возможность динамического создания PersistentVolume (PV) с различными характеристиками хранения. StorageClass позволяет администраторам Kubernetes абстрагировать и автоматизировать управление хранилищем, предоставляя пользователям возможность запрашивать хранилище с определенными свойствами без необходимости непосредственного взаимодействия с конкретными системами хранения.
- Поддерживает различные облачных провайдеров (Azure,GCP)
- Позволяет управлять политиками доступа при монтировании в контейнер
- Позволяет управлять **reclaimPolicy**(поведение при удалении)


---
### **ConfigMap**
- Пример yaml
```yaml
 apiversion: v1
 kind:ConfigMap
 metadata:
 	name:configmap-name
 data:
 	data.conf: |
 	server{
 		listen 80 default_server;
 		location /healthy{
 			return 200;
 		}
 	}
```
- Пример задания
	1. на уровне с resources
	2. на уровне с containers
```yaml
 	
 	volumeMounts:
 	- name: config-nginx
 	   mountPath: /etc/nginx/conf.d/
 ```
	 
```yaml
 	 volumes:
 	 - name: config
 	    configMap:
 		    name:configmap-name
```
- При монтировании нового файла из Мапа внутрь контейнера монтируется новый каталог с именем ,отображающим новую дату и меняет файл со старого на новый,старую удаляет 
- Способы монтирования
	- Volumes
	- ConfigMapKeyRef: (По аналогии с Secret)
### **Secret**
- Пример задания
```yaml
 	 env:
 	 - name: value
 	 valueFrom:
 		secretKeyRef:
 			name: secret-nane
 			key: key-name
```

- Кодируются в Base64 ,==НО не шифруются==
	- Виды
		- generic
			- Нужен для паролей/токенов приложений
		- docker-registry
			 - Данные авторизации для приватных registry(gitlab registry/docker hub/etc.)
			- Содержит адрес логин и т.д.
		- tls
			- tls сертификаты  для ingress контроллеров
			 - ключ и сам сертификат

- **Отличие от ConfigMap**
	- ==Данные хранятся в закодированном, но не зашифрованном  виде==
	- ==Нужен для хранения потенциально чувствительной информации==
	- ==Доступ к ним удобно ограничивать через RBAC==
- Vault
- Rbac
    - RoleBinding
    - ServiceAccount
    - Чёт ещё 
 - Operator
	- Под,поддерживает состояние системы в описанном состоянии
## **Работа с памятью и процессором в кластере**
![[resources-requests.jpg]]
### Примеры
#### Контейнер
```
containers:
- name: app-nginx
  image: nginx
  resources:
    requests:
      memory: 1Gi
    limits:
      cpu: 200m
```
#### Namespace
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: nxs-test
spec:
  hard:
    requests.cpu: 300m
    requests.memory: 1Gi
    limits.cpu: 700m
    limits.memory: 2Gi
```
### Limits
- Максимально возможная граница для приложения, сколько ресурсов ему можно использовать 
- При достижении лимита убьет приложение  по ==OOM(Out of memory)== с помощью ==OOM Killer==
- По памяти пишем в формате **512Mi** **1Gb**
- Для CPU пишем в форматах **1** ,если нужен целый процессор, или **100m(milicpu)**,где каждый процессор поделен на *1000m*
### Requests
- Количество ресурсов, которые необходимы приложению для его корректной работы 
-  По памяти пишем в формате **512m**,соответственно выделяем в мегабайтах
- Для CPU пишем в форматах **1** ,если нужен целый процессор, или **100m(milicpu)**,где каждый процессор поделен на *1000m*
### **LimitRange** 
- описывает политику ограничения на уровне контейнера/пода в ns и нужна для того, чтобы описать дефолтные ограничения на контейнер/под, а также предотвращать создание заведомо жирных контейнеров/подов (или наоборот), ограничивать их количество и определять возможную разницу значений в limits и requests
### **ResourceQuotas** 
- описывают политику ограничения в целом по всем контейнерам в ns и используется, как правило, для разграничения ресурсов по окружениям (полезно, когда среды жестко не разграничены на уровне нод)
### Master apiserver и лимиты
- Не имеет данных по **requests** и **limits**
- Есть проблема бесконечного рестарта без корректного определения лимитов==,включая master компоненты==
### QoS(quality of service)
- Назначется ресурсу в зависимости от его ограничений 
- **guaranuted** — назначается тогда, как для каждого контейнера в поде для memory и cpu задан request и limit, причем эти значения должны совпадать
- **burstable** — хотя бы один контейнер в поде имеет request и limit, при этом request < limit
- **best effort** — когда ни один контейнер в поде не ограничен по ресурсам
	- ==при одинаковом приоритете, kubelet в первую очередь будет выселять с ноды поды с QoS-классом best effort.==
## Проверка работоспособности приложения (Probe)
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

```
startupProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
  failureThreshold: 30
```
## **На почитать**
- ЕБАНЫЙ. Vault
	-  Vault bank ,bank checker
- Istio
- Argocd
### **Kubectl**
![[kubectl-cheat-sheet.png]]
- можно писать цепочки вида -f file.yaml -f file2.yaml
- kubectl get  ep (получить группы endpoints для селектора сервиса)
### **Helm chart**
-  Что то поподробнее, но пока хз что 
- Pod affinity
- Network policy
- Одуплиться с GO
- Network Policy
- Как ноды куба взаимодейтствуют друг с другом
	
