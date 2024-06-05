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
- Назначается ресурсу в зависимости от его ограничений 
- **guaranuted** — назначается тогда, как для каждого контейнера в поде для memory и cpu задан request и limit, причем эти значения должны совпадать
- **burstable** — хотя бы один контейнер в поде имеет request и limit, при этом request < limit
- **best effort** — когда ни один контейнер в поде не ограничен по ресурсам
	- ==при одинаковом приоритете, kubelet в первую очередь будет выселять с ноды поды с QoS-классом best effort.==