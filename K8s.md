![](https://k8sdesiredstate.github.io/k8s_logo.png)
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
- Operator
	- Под,поддерживает состояние системы в описанном состоянии
## **Масштабирование внутри кластера**
- HPA 
- VPA 
- PBD 
## **Ресурсы кластера**
- Deployment
- StatefulSet
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
## **Работа с памятью и процессором в кластере**
- REQUESTS 
- LIMITS 
     - И то и то для памяти и для процессора и как делится ядро 
     - Что куда и как работает потыкать
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
	
