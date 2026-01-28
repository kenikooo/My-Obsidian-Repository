— Перейти в [[HUB]] —
> [!BUG] **Задание выполнено не до конца**
### ЗАДАНИЕ:
**Часть 1:**

- Изучите предложенный материал в рамках темы: "Кластер высокой доступности Kubernetes в ОС 'Альт'"
1. Стековая (составная) топология etcd ([ссылка](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-ha-cluster.html#kubernetes-stacked-etcd-topology))
2. Внешняя etcd-топология ([ссылка](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-external-etcd-topology.html))
3. Создание HA-кластера с помощью kubeadm ([ссылка](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-ha-cluster-kubeadm.html))
- Стековая (составная) топология etcd ([ссылка](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-ha-cluster-kubeadm.html#kubernetes-stacked-etcd-topology-kubeadm))
- Внешняя etcd-топология ([ссылка](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-external-etcd-topology-kubeadm.html))

**Часть 2:**
- Попытайтесь развернуть 2 независимых друг от друга кластера высокой доступности Kubernetes в ОС "Альт":
1. со стековой топологией (узлы etcd размещаются вместе с узлами плоскости управления);
2. с внешней etcd-топологией (etcd работает на узлах, отдельных от плоскости управления).

> [!INFO] Внешний Офис
| Имя хоста    | Адрес        |
| ------------ | ------------ |
| Master-Node1 | 192.168.0.2  |
| Master-Node2 | 192.168.0.3  |
| Master-Node3 | 192.168.0.4  |
| Worker-Node1 | 192.168.0.10 |
| Worker-Node2 | 192.168.0.11 |
| Worker-Node3 | 192.168.0.12 |
| Worker-Etcd1 | 192.168.0.20 |
| Worker-Etcd2 | 192.168.0.21 |
| Worker-Etcd3 | 192.168.0.22 |

> [!INFO] Внутренний офис
| Имя хоста    | Адрес        |
| ------------ | ------------ |
| Master-Node1 | 192.168.0.5  |
| Master-Node2 | 192.168.0.6  |
| Master-Node3 | 192.168.0.7  |
| Worker-Node1 | 192.168.0.13 |
| Worker-Node2 | 192.168.0.14 |
| Worker-Node3 | 192.168.0.15 |

> [!WARNING] ПРЕДУПРЕЖДЕНИЕ
> Если вдруг начались проблемы с кластером, вы можете сбросить выполнил следующие команды на каждой машине из кластера (ПОЛНОСТЬЮ СНОСИТ ВСЕ НАСТРОЙКИ):
> * kubeadm reset --force
> * rm -rf /etc/kubernetes/ /var/lib/etcd/ ~/.kube/
> * crictl rm -fa
> * systemctl restart {crio,kubelet}

Для удобства делаем проброс порта через VirtualBox, по следующему пути:
```bash
Сеть -> Сеть NAT (создаем сеть, например, с адресом 192.168.0.0/24), переходим во вкладку "Проброс портов", IPv4 и справа жмем "+":
- Имя - любое
- Протокол - TCP
- Адрес хоста - IPv4-адрес Вашего рабочего компьютера (можно узнать запустив терминал/командную строку и прописав: ipconfig /all или ip -c a)
- Порт хоста - 2222 (к каждому новому правилу для ВМ +1, тоесть следующий будет: 2223)
- Адрес гостя - IP-адрес нашей виртуальной машины
- Порт гостя - стандратный, если не менялся: 22
Пример:
* Master-Node1 | TCP | 192.168.15.210 | 2222 | 192.168.0.2 | 22
* Master-Node2 | TCP | 192.168.15.210 | 2223 | 192.168.0.3 | 22
...
* Worker-Node1 | TCP | 192.168.15.210 | 2225 | 192.168.0.10 | 22 
После создания, сохраняем
```
Подключаемся следующей командой, из терминала/командной строки, с рабочего компьютера (хоста):
```bash
ssh -p 2222 user@192.168.15.210
где:
* -p - подключение происходит по порту
* 2222 - порт, по которому мы подключаемся
* user - имя пользователя, под которым мы будем подключаться к ВМ (может отличаться)
* 192.168.15.210 - адрес хоста, потому что мы делали проброс по порту, и ссылаясь на порт, он отправит нас на привязанный к порту адрес
```

## Настройка внутреннего офиса: на всех 6 машинах
```bash
# Обязательно отключаем SWAP:
> swapoff -a

# Устанавливаем пакеты:
> apt-get update && apt-get install -y kubernetes1.33-kubeadm kubernetes1.33-kubelet kubernetes1.33-crio

# Запускаем службы:
> systemctl enable --now {crio,kubelet}
```

### Инициализация кластера на Master-Node1
Если у Вас установлен Docker/ContainerD, удалим его, а иначе кластер не создаться:
```bash
apt-get remove docker containerd -y
```
Создание кластера:
```bash
# Создаем кластер
> kubeadm init \
>  --control-plane-endpoint "192.168.0.100:6443" \
>  --pod-network-cidr=192.168.0.0/16 \
>  --upload-certs
  
# Настраиваем kubectl
> mkdir -p /home/user/.kube
> cp -i /etc/kubernetes/admin.conf /home/user/.kube/config
> chown user:user /home/user/.kube/config
```
Если после создания выводится ошибка, в которой есть строка:
```bash
Once you have found the failing container, you can inspect its logs with:
- 'crictl --runtime-endpoint unix:///var/run/crio/crio.sock logs CONTAINERD'
# Исправляем:
> crictl ps | grep kube # если есть запущенные службы kube-apiserver и kube-controller-manager - отключаем
> crictl stop <ID_службы> # вводим цифры-буковки из первого столбца
```
### Присоединение узлов
На Master-Node2 и Master-Node3, выполняем команду из вывода kubeadm init (на Master-Node1):
```bash
> kubeadm join 192.168.0.100:6443 \
>  --token <token> \
>  --discovery-token-ca-cert-hash <hash> \
>  --control-plane \
>  --certificate-key <key>
```

На Worker-Node1, Worker-Node2 и Worker-Node3, вторую команду из вывода kubeadm init (на Master-Node1):
```bash
> kubeadm join 192.168.0.100:6443 \
>  --token <token> \
>  --discovery-token-ca-cert-hash <hash>
```

### Установка CNI (Calico):
На любом Master узле:
```bash
> kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
> kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
```

### Настройка внешнего офиса: на всех 9 машинах
```bash
swapoff -a
apt-get update && apt-get install -y kubernetes1.33-kubeadm kubernetes1.33-kubelet kubernetes1.33-crio
systemctl enable --now {crio,kubelet}
```

Настройка выделенного ETCD кластера, на Worker-Etcd1(192.168.0.20):
```bash
# Создаем конфиг для kubeadm
vim /etc/kubernetes/kubeadmcfg.yaml
# Содержимое:
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "192.168.0.20"
nodeRegistration:
  criSocket: "unix:///var/run/crio/crio.sock"
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
etcd:
  local:
    serverCertSANs:
    - "192.168.0.20"
    - "192.168.0.21" 
    - "192.168.0.22"
    peerCertSANs:
    - "192.168.0.20"
    - "192.168.0.21"
    - "192.168.0.22"

# Запускаем etcd
kubeadm init phase etcd local --config=/etc/kubernetes/kubeadmcfg.yaml
```
Повторить на Worker-Etcd2 и Worker-Etcd3 с соответствующими IP-адресами.

### Инициализация Control Plane с внешним ETCD
На Master-Node1 (192.168.0.2):
```bash
# Создаем конфиг для kubeadm
vim kubeadm-config.yaml
# Содержимое
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.31.0
controlPlaneEndpoint: "192.168.0.200:6443"
networking:
  podSubnet: "10.244.0.0/16"
etcd:
  external:
    endpoints:
    - "https://192.168.0.20:2379"
    - "https://192.168.0.21:2379"
    - "https://192.168.0.22:2379"
    caFile: /etc/kubernetes/pki/etcd/ca.crt
    certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
    keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key

# Инициализируем кластер
kubeadm init --config=kubeadm-config.yaml --upload-certs

# Настраиваем kubectl
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

Присоединяем остальные узлы, Master-Node2 и Master-Node3:
```bash
# Используем команду из вывода kubeadm init с флагом --control-plane
kubeadm join 192.168.0.200:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash> \
  --control-plane \
  --certificate-key <key>
```

На Worker-Node1, Worker-Node2, Worker-Node3:
```bash
# Используем команду из вывода kubeadm init БЕЗ флага --control-plane
kubeadm join 192.168.0.200:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash <hash>
```

На любом Master-узле второго кластера:
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml

# Для второго кластера используем другой CIDR
vim custom-resources.yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.245.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
---
apiVersion: operator.tigera.io/v1
kind: APIServer 
metadata:
  name: default
spec: {}

kubectl apply -f custom-resources.yaml
```