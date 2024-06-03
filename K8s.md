![](https://k8sdesiredstate.github.io/k8s_logo.png)

---
## **Компоненты кластера**
![[components-of-k8s.png]]
[[K8s components]]
#### Kubernetes работает по pull системе, все работает за счет подписок на события и вызовов одного компонента другим 
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
### PV/PVC/PV Provisioner/StorageClass
[[K8s PV-PVC]]
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
### Rbac
- RoleBinding
- ServiceAccount
- Чёт ещё 

### Operator
	- Под,поддерживает состояние системы в описанном состоянии
## **Работа с памятью и процессором в кластере**
[[K8s mem cpu  managment]]
## Проверка работоспособности приложения (Probe)
[[K8s probe]]
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
	
