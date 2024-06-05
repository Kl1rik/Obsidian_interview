#k8s
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