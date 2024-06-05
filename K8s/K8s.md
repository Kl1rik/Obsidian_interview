![](https://k8sdesiredstate.github.io/k8s_logo.png)

---
## **Компоненты кластера**
![[components-of-k8s.png]]
[[K8s components]]
#### Kubernetes работает по pull системе, все работает за счет подписок на события и вызовов одного компонента другим 
## **Масштабирование внутри кластера**
### HPA 
![[k8s-hpa.png]]
- Отвечает за масштабирование реплик подов
- Работает на основании метрик, основная цель -- ответ на возросшую на приложение нагрузку
- ==По умолчанию погрешность 10%==
- Хорошо работает вместе с **PBD**
#### Схема работы
1. HPA непрерывно проверяет значения метрик, указанные при установке, с интервалом по умолчанию 30 секунд.  
    
2. HPA пытается увеличить количество модулей, если достигнут заданный порог.  
    
3. HPA обновляет количество реплик внутри контроллера развертывания/репликации.  
    
4. Контроллер развертывания/репликации затем разворачивает все необходимые дополнительные модули.
5. После спада нагрузки контроллер убивает все модули и приводит кластер к исходному состоянию
#### За чем следить
- Переопределение **delay** ==по умолчанию== по необходимости (==3 минуты==),лучше поменять если есть проблемы с сетью / настройкой
- Интервал проверки по умолчанию 30 секунд
- По умолчанию ждет стабилизации 5 минут
- ==При оценке времени масштабирования имейте в виду худший и лучший сценарии.==
#### Пример yaml
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
### VPA 
![[k8s-vpa.png]]
- Отвечает за масштабирование количество ресурсов пода(**Жизненно необходимо для контроля нагрузки на БД**)
- Работает на основе метрик
#### Схема работы 
1. VPA непрерывно проверяет значения метрик, указанные при установке, с интервалом по умолчанию 10 секунд.  
    
2. Если достигнут заданный порог, VPA пытается изменить выделенное количество ресурсов.  
    
3. VPA обновляет количество ресурсов внутри контроллера развертывания/репликации.  
    
4. При перезапуске модулей все новые ресурсы применяются к созданным инстансам.

#### За чем следить
 - у VPA есть интересная функция рекомендаций (**VPA Recommender**). Она отслеживает использование ресурсов и события OOM всех модулей, чтобы предложить новые значения памяти и процессорного времени на основе интеллектуального алгоритма с учетом исторических метрик.
- ==**VPA Recommender** не отслеживает «лимит» ресурсов==. Это может привести к тому, что модуль монополизирует ресурсы внутри узлов. Лучше установить предельное значение на уровне пространства имен, чтобы избежать огромного расхода памяти или процессорного времени.
- ==При оценке времени масштабирования имейте в виду худший и лучший сценарии.==
#### Пример yaml
```yaml
apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: hamster-vpa
spec:
  # recommenders field can be unset when using the default recommender.
  # When using an alternative recommender, the alternative recommender's name
  # can be specified as the following in a list.
  # recommenders: 
  #   - name: 'alternative'
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: hamster
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 100m
          memory: 50Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]
```

### PBD 
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
	