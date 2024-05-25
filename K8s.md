![[k8s-cluster.png]]

## **Компоненты кластера**
- Control plane
	- Компоненты
		- Kube-scheduler
		- Api-gateway
		- Controller
			- Cluster-manage
			- Cloud-manage
		- Etcd
- Operator
	- Под,поддерживает состояние системы в описанном состоянии
## **Масштабирование внутри кластера**
- HPA 
- VPA 
- PBD 
## **Записи ,роли , секреты внутри кластера**
- StatefulSet
- PV
- PVC
- SECRETS 
    - В чем отличия от конфигмапы
- CONFIGMAP
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
- Helm chart
     -  Что то поподробнее, но пока хз что 
- Pod affinity
- Network policy
- Одуплиться с GO
- Network Policy
- Как ноды куба взаимодейтствуют друг с другом
	