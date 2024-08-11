---
share_link: https://share.note.sx/m0r0ti99#uAlviHotPHNqXaPWvGmou3luocEOJZm4Jn2Wa59kkRY
share_updated: 2024-06-23T16:29:04+03:00
---
![](https://k8sdesiredstate.github.io/k8s_logo.png)
#k8s
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
- Хорошо работает вместе с **[[#PBD]]**
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
 - у VPA есть интересная функция рекомендаций (**VPA Recommender**). Она отслеживает использование ресурсов и события [[K8s mem cpu  managment#OOM|OOM]](out of memory) всех модулей, чтобы предложить новые значения памяти и процессорного времени на основе интеллектуального алгоритма с учетом исторических метрик.
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
	- Blue/Green (две среды с разными версиями проги )
	- Canary (сначала маленькой группе пользователей новую версию,потом остальным)
### StatefulSet
- Поддерживает PVC
- В отличие от Deployment не изменяется динамически
- Необходим для приложений ,для которых нужно сохранять состояние 
### PV/PVC/PV Provisioner/StorageClass
[[K8s PV-PVC]]
### ConfigMap/Secret
[[K8s configmap & Secret]]
- Vault
### Rbac
![[Pasted image 20240615143223.png]]
#### Role
- Список в YAML манифесте ,**описывающий ,но не назначающий права на объекты в кластере**
##### Subjects
- пользователи(группы пользователей )
```yaml
kind: Role  
apiVersion: rbac.authorization.k8s.io/v1  
metadata:  
namespace: mynamespace  
name: example-role  
rules:  
- apiGroups: [""]  
resources: ["pods"]  
verbs: ["get", "watch", "list"]

```
#### RoleBinding

>  definition of what Subjects have which Roles

- Связывает **Role** и **ServiceAccount** в **определенном** **namespace**
```yaml
kind: RoleBinding  
apiVersion: rbac.authorization.k8s.io/v1  
metadata:  
name: example-rolebinding  
namespace: mynamespace  
subjects:  
- kind: User  
name: example-user  
apiGroup: rbac.authorization.k8s.io  
roleRef:  
kind: Role  
name: example-role  
apiGroup: rbac.authorization.k8s.io
```
#### ClusterRole

#### ClusterRoleBinding
#### ServiceAccount

### Operator
	- Под,поддерживает состояние системы в описанном состоянии

## Паттерны в подах
- В поде можно и иногда нужно использовать несколько типов контейнеров 
### Основные паттерны **Pod** 
#### (для существующих приложений/legacy)
#### Паттерны **Adapter** и **Ambassador**
![[Pod-pattern.png]]
#### Паттерн **Sidecar**
![[pod-pattern-sidecar.png]]
- Нужен в случаях, когда необходима:
	- **Инъекция** **секретов**
	- **Динамическое обновление ConfigMap**
	- **Кэширование**
	- **Передача логов в stdout**
#### Паттерн **Init**
![[pod-pattern-init.png]]
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
### Плейлисты 
#### Слёрм академия
https://www.youtube.com/live/Jp866ltZBSk?si=GgNxosjzXamq5Jqi
#### ADV-IT
https://www.youtube.com/watch?v=q_nj340pkQo&list=PLg5SS_4L6LYvN1RqaVesof8KAf-02fJSi&pp=iAQB