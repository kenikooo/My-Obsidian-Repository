— Перейти в [[HUB]] —
>[!SUCCESS] **Задание выполнено и работает корректно**

> [!INFO] **Дистрибутивы, которые потребуется установить на MASTER-NODE1 и WORKER-NODE1/2 для выполнения работы**
> - kubernetes1.33-kubeadm
> - kubernetes1.33-kubelet
> - kubernetes1.33-crio
> - crio-tools1.33

> [!QUESTION] **Используемые ОС**
> `ALT-JeOS-p11`
> - MasterNode1
> - WorkerNode1 
> - WorkerNode2

> [!QUOTE] IP-Адреса
> - MasterNode1: **192.168.0.10/24**
> - WorkerNode1: **192.168.0.12/24**
> - WorkerNode2: **192.168.0.13/24**

После установки, запускаем crio и kubelet:
```bash
systemctl enable --now {crio,kubelet}
```
Также требуется отключить файл подкачки для выполнения последующих шагов в работе:
```bash
swapoff -a
```
В самом начале создаем директорию для дальнейшей работы на **MASTER-NODE1**:
```bash
mkdir ./nginx-test # Создание директории в текущей
cd !$ # Переход в созданную директорию
touch {deployment.yaml,index.html} # Создаем deployment и index (сайт)
```
Развернем Calico:
```bash
> kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/tigera-operator.yaml 
> kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/master/manifests/custom-resources.yaml
```

> [!WARNING] Содержимое deployment.yaml
> ```yaml
> apiVersion: app/V1
> kind: Deployment
> metadata:
> 	name: nginx-deployment
> spec:
> 	selector:
> 		matchLabels:
> 			app: nginx-app
> 	template:
> 		metadata:
> 			labels:
> 				app: nginx-app
> 		spec:
> 			containers:
> 			- name: nginx
> 			   image: nginx
> 			   ports:
> 			   - containerPort: 80
> 			   volumeMounts:
> 			   - name: homepage-volume
> 			    configMap:
> 				    name: homepage
> ---
> apiVersion: v1
> kind: Service
> metadata:
> 	name: nginx-service
> spec:
> 	type: NodePort
> 	selector:
> 		app: nginx-app
> 	ports:
> 	   - protocol: TCP
> 	    port: 80
> 	    targetPort: 80
> ```

> [!BUG] Содержимое index.html
> ```
> <h1>YOUR NAME HERE</h1>
> ```

Для начала создадим configMap:
```bash
kubectl create configMap homepage --from-file=index.html # Создаем объект для хранения конфигурационных данных в виде пар "ключ-значение"
```
После этого мы можем приступить к развертыванию сервиса:
```bash
kubectl create -f deployment.yaml # Разворачиваем deployment и service's
```
Выводим информацию о развернутых сервисах и узнаем порт:
```bash
kubectl get service,deployment,pods

> service # Выводит информацию о развернутых сервисах, нам нужен сервис с названием service/nginx-service, смотрим на PORT 80:31312 - наш сервис развернут на 31312 порту

> deployment # выведет информацию о развернутом deployment'е

> pods # Поды, которые развернулись
```
Если нам нужно узнать на какой именно машине развернулись сервисы используется следующая команда: 
```bash
kubectl get service,deployment,pods -o wide # Флаг -o wide выведет дополнительную информацию, тоесть полученный результат будет более обширен, мы увидим ноды на которых развернуты сервисы.
```
Чтобы узнать какие поды запущены в общем (и на какой ноде), используется следующая команда: 
```bash
kubectl get pods -A -o wide # Флаг -A выведет абсолютно все сервисы, те которые развернули мы deployment'ом и те, что нужны для работы кластера в целом
```
Из вывода, мы выделим следующие:
```bash
> default nginx-deployment-ID-ID 1/1 RUNNING .. worker-node1 # Наш deployment который прилетел на первого воркера
> tigera-operator tigera-operator-ID-ID 1/1 RUNNING .. worker-node2 # Calico работающий на второй воркер ноде
```
