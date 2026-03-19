— Перейти в [[HUB]] —
> [!BUG] **Заметка не дописана до конца**

> [!INFO] Таблица адресации
>| Адрес           | Имя-Узла     | Роль         |
>| --------------- | ------------ | ------------ |
>| 192.168.0.2/24  | Master-Node1 | ControlPlane |
| 192.168.0.3/24  | Master-Node2 | ControlPlane |
| 192.168.0.4/24  | Master-Node3 | ControlPlane |
| 192.168.0.10/24 | Worker-Node1 | None         |
| 192.168.0.11/24 | Worker-Node2 | None         |
| 192.168.0.12/24 | Worker-Node3 | None         | 

> **Используемые дистрибутивы**
> - kubernetes1.33-kubeadm
> - kubernetes1.33-kubelet
> - kubernetes1.33-crio
> - cri-tools1.33

В самом начале обязательно запускаем службы crio и kubelet:
```bash
systemctl enable --now crio
systemctl enable --now kubelet
```

Для создания кластера обязательно должно быть несколько машин, одна из которых будет управляющим узлом: 3 Управляющих узла и 3 (или более) воркер.

>[!BUG] Системные требования для развертывания кластера
> - ОЗУ: 2гб или более на узел;
> - Процессор: 2 ядра или более;
> - Сетевая доступность между всеми узлами;
> - Возможность разрешения имен (через DNS или /etc/hosts);
> - Swap должен быть отключен.

Отключаем swap:
```bash
swapoff -a
```

Последующая работа будет проходить на главном узле (Master-node1).
```bash
kubeadm init --control-plane-endpoint 192.168.0.2:6443 --upload-certs --pod-network-cidr=10.244.0.0/16 # где 192.168.0.2 - адрес главного узла, а --pod-network-cidr=10.244.0.0/16 - адрес внутренней сети *рекомендуется оставлять данное значение для правильной работы flannel
```
Когда кластер успешно развернется в конце мы увидим токены, записываем две выданных команды, они пригодятся в дальнейшем для присоединения менеджеров и воркеров.

Настроим Kubernetes для работы от обычного пользователя:
```bash
- Заходим под обычного Пользователя (на каждой ВМ):
$ mkdir ~/.kube
- Возвращаемся в рута и прописываем следующие команды:
cp /etc/kubernetes/admin.conf /home/<Пользователь>/.kube/config
chown <Пользователь>: /home/<Пользователь>/.kube/config
- Выходим из под рута и заходим по обычным Пользователем, для установки flannel:
$ kubectl apply -f https://gitea.basealt.ru/alt/flannel-manifests/raw/branch/main/p11/latest/kube-flannel.yml
```
После установки flannel, проверим состояние компонентов (все должны находится в состоянии READY (1/1) и STATUS - Running.
```bash
$ kubectl get pod -n kube-system -w
```
Далее, на остальных узлах, мы прописываем команды скопированные раннее, первую используем на МЕНЕДЖЕРАХ, вторую (покороче, она без --control-plane, --certificate-key) - на обычных воркерах.

После успешного присоединения в конце будет написано Run 'kubectl get nodes' on the control-plane to see this node join the cluster. Заходим на главный узел и прописывает команду:
```bash
kubectl get nodes
```

* Если при присоединении узлов к главному узлу вылезают ошибки, применяем следующие команды: 
Генерация нового токена и вывод команды для присоединения узла (выполняется на главном узле):
```bash
kubeadm token create --print-join-command
```

Выдача роли:
```bash
kubectl label node <Имя-узла> node-role.kubernetes.io/control-plane=""
```

Используемая документация:
1. https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-cluster.html
2. https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/kubernetes-ha-cluster-kubeadm.html