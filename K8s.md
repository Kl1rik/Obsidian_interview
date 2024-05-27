![[k8s-cluster.png]]

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
- PV
- PVC
### **ConfigMap**
- Пример yaml
> 	---
> 	apiversion: v1
> 	kind:ConfigMap
> 	metadata:
> 		name:configmap-name
> 	data:
> 		data.conf: |
> 	 server{
> 		 listen 80 default_server;
> 		 location /healthy{
> 			 return 200;
> 		 }
> 	 }
- Пример задания
> 	на уровне с resources
> 	volumeMounts:
> 	- name: config-nginx
> 	   mountPath: /etc/nginx/conf.d/
> 	на уровне с containers
> 	 volumes:
> 	 - name: config
> 	    configMap:
> 		    name:configmap-name
- При монтировании нового файла из Мапа внутрь контейнера монтируется новый каталог с именем ,отображающим новую дату и меняет файл со старого на новый,старую удаляет 
- Способы монтирования
	- Volumes
	- ConfigMapKeyRef: (По аналогии с Secret)
### **Secret**
- Пример задания
> 		env:
> 	 - name: value
> 	 valueFrom:
> 		secretKeyRef:
> 			name: secret-nane
> 			key: key-name


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
- Kubectl
	- все что может почитать 
	- можно писать цепочки вида -f file.yaml -f file2.yaml
- Helm chart
     -  Что то поподробнее, но пока хз что 
- Pod affinity
- Network policy
- Одуплиться с GO
- Network Policy
- Как ноды куба взаимодейтствуют друг с другом
	
