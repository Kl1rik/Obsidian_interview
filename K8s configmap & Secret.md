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