
1. **Control Plane Internals:**
    - [x] **Scheduler:** 
	    - [x] Как он выбирает node'у для pod'ы? (Предикаты -> Приоритеты). 
	    - [x] Как работают `nodeAffinity`, `podAffinity` и `taints/tolerations` на уровне кода?
	    - [x] Что такое Scheduling Framework?
	    Ожидаемый результат: конспект на тему **"Sсheduler"**

    - [x] **Controller Manager:** 
	    - [x] Что такое контроллеры и цикл контроля (reconciliation loop)?
	    - [x] Каскадная реакция (Deployment + ReplicaSet)
	    Ожидаемый результат: конспект на тему **"Controller Manager"**

    - [ ] **ETCD:** 
	    - [ ] Зачем нужны snapshots? 
	    - [ ] Что такое "quorum"? 
	    - [ ] Почему etcd любит быстрые диски (SSD)?
	    - [ ] Watch-механизм
	    Ожидаемый результат: конспект на тему **"ETCD"**

    - [ ] **API-Server:** 
	    - [ ] Что за сущность и зачем нужен кластеру?
	    - [ ] AdmissionWebhooks + разбор нескольких стандартных Admission Contollers (например, `NamespaceLifecycle`, `ResourceQuota`)
	    Ожидаемый результат: конспект на тему **"API-Server"**

	- [ ] **Kubelet:**
		- [ ] ==план по kubelet==

    - [ ] Полный цикл процессов при использовании `kubectl apply -f`
		Ожидаемый результат: схема с одноименным названием

2. **Network:**
    - [ ] Как работает **kube-proxy**? 
	    - [ ] В чем разница между режимами `userspace`, `iptables` и `ipvs`?
		Ожидаемый результат: конспект на тему **"Kube-proxy"**

    - [ ] Полный цикл настройки сети под капотом в кластере k8s
		Ожидаемый результат: схема с одноименным названием

    - [ ] **CNI (Container Network Interface):** 
	    - [ ] Зачем нужны сетевые плагины? 
	    - [ ] Как вообще под получает IP-адрес?
	    - [ ] veth pair, мост

    - [ ] **Network Policies:** 
	    - [ ] Как Calico/Cilium перехватывает трафик на уровне ядра + в чем разница между двумя подходами (eBPF, iptables) ?
		Ожидаемый результат: конспект на тему **"Сетевые плагины на примере Calico/Cilium"**

3. **Security:**
    - [ ] **RBAC:**
	    - [ ] Как составить политику так, чтобы приложение видело только свои секреты?
	    Ожидаемый результат: конспект на тему **"RBAC"**

    - [ ] **ServiceAccounts:** 
	    - [ ] Чем отличается от обычного пользователя?
	    - [ ] Что происходит при создании SA
	    Ожидаемый результат: конспект на тему **"Пользователи в кластере k8s"**

    - [ ] **PodSecurity Standards (бывший PSP):** 
	    - [ ] Как запретить запуск контейнеров от root?
	    Ожидаемый результат: конспект на тему **"Pod Security"**

4. **Хранение данных (Storage):**
	- [ ] **Persistent Volumes (PV) и Persistent Volume Claims (PVC):** 
		- [ ] Как они связываются? 
		- [ ] Что такое dynamic provisioning (StorageClass)?
		Ожидаемый результат: конспект на тему **"PV + PVC + SC"**

	- [ ] **CSI (Container Storage Interface):** 
		- [ ] Как работают драйверы для облаков (AWS EBS, GCE PD) и on-premise (NFS, Rook/Ceph)?
		Ожидаемый результат: конспект на тему **"CSI"**

	- [ ] **StatefulSets:** 
		- [ ] Чем отличаются от Deployment?
		- [ ] Зачем нужен стабильный network identity (DNS) и стабильный storage? (Для баз данных).
		Ожидаемый результат: конспект на тему **"PV + PVC + SC"**

	- [ ] **Типы Volume:** Разница между `emptyDir`, `hostPath`, `configMap` и `persistentVolumeClaim`
		Ожидаемый результат: конспект на тему **"Volumes"**

5. **Kubespray:** 
	- [ ] Развернуть работающий кластер с помощью kubespray
	    Ожидаемый результат: работающий кластер с несколькими инстансами различных Controller'ов и сетевым плагином (Calico/Cilium) 