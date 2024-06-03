### Control plane
#### Kube-scheduler
- ==Назначает PODы на нужные ноды==
- Учитывает QoS,requests,pod Affinity/Anti-affinity,Priority class
#### Api-server
- ==Центральный компонент Kubernetes==
- Единственный общается с etcd
- Работает по REST API
- Регулирует доступ через аутентификацию и авторизацию 
#### Controller Manager
- ==Работает над созданием ресурсов и их своевременным удалением ==
- Работает с Node/Replication/Endpoints/Daemonset/... controller 			
- Работает с GarbageCollector
#### Etcd
- [[etcd.canvas|etcd]]
- ==Хранилище кластера==
- Хранятся все секреты, манифесты и т.д.
- управляется etcdctl
- ==порт 2379 -- для клиентов ,2380 -- для работы хранилищ между собой==

### Node/ Worker node(s)
#### Kubelet
- ==Systemd служба==
- Работает на каждой ноде
- ==Отдает команды Docker daemon==
- ==Создает поды==
- Делает **Probe**
##### ==Максимальное количество подов на узле : 110==