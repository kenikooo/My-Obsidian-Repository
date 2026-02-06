— Перейти в [[HUB]] —
> [!INFO] Что такое **NFS**?
Общий доступ к файлам и каталогам по сети**

> [!WARNING] Используемые дистрибутивы **nfs-utils**, **nfs-server**

Устанавливаем необходимые дистрибутивы:
```bash
apt-get update && apt-get install nfs-server nfs-utils -y
```
Запускаем службу:
```bash
systemctl enable --now nfs
```

Для начала создадим каталог, который будет общедоступным:
```bash
> mkdir -p /opt/share # Создаем несколько директорий при помощи ключа -p
> chmod 777 /opt/share # Задаем права на Чтение-Запись-Исполнение
```

Отредактируем файл, по пути /etc/exports:
```bash
/opt/share *(rw,sync,no_subtree_check) # '*' разрешаем любому устройству обнаруживать нашу общедоступную директорию, но можно написать адрес вместо '* 'к примеру 192.168.100.0/24
```

Применяем изменения:
```bash
> exportfs -ra
> systemctl restart nfs-server
> systemctl enable --now nfs-server
```

#### Подключение директории на клиентах
---
> [!QUESTION] ALT Linux

Устанавливаем дистрибутив:
```bash
> apt-get install nfs-common -y
```

Создадим точку монтирования:
```bash
> mkdir -p /mnt/adminshare # Точка монтирования может называться иначе
```

Добавляем запись в **_/etc/fstab_**:
```bash
192.168.100.2:/opt/share /mnt/adminshare nfs defaults,_netdev 0 0
```

Монтируем:
```bash
> mount -a
```

Проверяем:
```bash
> df -h | grep adminshare
```
---
> [!INFO] Windows 10 PRO

Для начала включим возможности NFS-Клиента, чтобы использовать команды для монтирования в **_CMD_** или **_PowerShell_**, по следующему пути:
```
| Панель управления
 | Программы
  | Программы и компоненты
   | Включение и отключение компонентов Windows
    | Служба для NFS (разворачиваем)
     | Клиент для NFS
```

Запустим командную строку **_Win+R_** -> **_cmd_**

Монтирование через "сеть" в "Мой компьютер":
```bash
\\АДРЕС_СЕРВЕРА\путь\к папке
# Пример:
\\192.168.10.3\srv\datanet
# Путь на Linux:
/srv/datanet
```

---
> [!BUG] Необязательно

Создадим пустую директорию, если ее нет:
```
> mkdir C:\adminshare
```

Можно создать символическую ссылку, до директории, в которой мы сможем создавать файлы и они будут появляться на диске, который является примонтированным:
```bash
> mklink /D C:\adminshare Z:\
```

Смонтируем NFS-Каталог прямо в эту папку:
```bash
> mount -o anon \\192.168.100.2\opt\share C:\adminshare
```
---
> [!WARNING] Обязательно, если пункт "Необязательно" был пропущен

Примонтируем NFS-Каталог:
```bash
> mount -o anon \\192.168.100.2\opt\share Z:
```

Если с примонтированным диском возникли проблемы, можно осуществить размонтирование, об этом - ниже.

---
> [!BUG] Размонтирование диска перед повторным монтированием net use Z: /delete

> [!EXAMPLE] Включение компонента командой
```bash
# Включаем NFS-клиент:
> Enable-WindowsOptionalFeature -Online -FeatureName ServicesForNFS-ClientOnly -NoRestart
```

> [!TODO] Автоматическое монтирование
```bash
net use Z: \\192.168.100.2\opt\share /persistent:yes
# Если требуется Логин:Пароль
net use Z: \\192.168.100.2\opt\share /user:имя_пользователя пароль /persistent:yes
# Если пользователь доменный важно указывать через DOMAIN\
net use Z: \\192.168.100.2\opt\share /user:DOMAIN\имя_пользователя пароль /persistent:yes
```
---

Владелец - admin_group, права для client_group через ACL
```bash
chown -R root:"HORNS\\admin_group" /srv/datanet
chmod 2770 /srv/datanet
setfacl -m g:"HORNS\\client_group":rwx /srv/datanet
setfacl -m d:g:"HORNS\\client_group":rwx /srv/datanet
```

Используемая документация:
- [NFS for ALT Linux 11.1](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/nfs.html)
