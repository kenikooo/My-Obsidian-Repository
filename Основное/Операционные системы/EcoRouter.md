— Перейти в [[HUB]] —
Содержание:
1. [[#Отключение 10-минутного вылета]]
2. [[#Установка пароля на Enable]]
3. [[#Шифрование паролей]]
4. [[#Создание/установка пароля для пользователя]]
5. [[#Сохранение конфигурации]]
6. [[#Установка баннера]]
7. [[#Настройка OSPFv2]]
8. [[#Создание/редактирование интерфейса]]
9. [[#Определение пула NAT]]
10. [[#Настройка DHCP]]

> [!info] Режимы
```bash
enable
configure terminal
````

> [!warning] Смена имени маршрутизатора
```bash
hostname <имя>
```

### Отключение 10-минутного вылета:

```bash
line console 0
exec-timeout 0
exit
```

```bash
line vty 0 871
exec-timeout 0
```

### Установка пароля на Enable:

```bash
enable secret P@ssw0rd  # Пароль для режима администрирования (enable)
```

### Шифрование паролей:

```bash
service password-encryption
```

### Создание/установка пароля для пользователя:

```bash
username <Имя>        # Создаем пользователя
password <пароль>     # Устанавливаем пароль
role admin            # Выдаем роль администратора
```

### Сохранение конфигурации:

```bash
write
```

> [!warning] Можно выполнять в любом режиме

### Установка баннера:

```bash
banner motd "Текст баннера"
```

---

### Настройка OSPFv2

```bash
router ospf <№>
	ospf router-id <IP>
	network <net_ip> <reverser> area <№>
```

Исключение интерфейсов из процесса OSPF:
```bash
ospf passive-interface <имя интерфейса>
```

Настройка аутентификации:
```bash
ip ospf authentication
area <№> authentication
```

Быстрый способ:
```bash
default passive-interface
no passive interface <имя интерфейса>
```

Настройка таймеров OSPF:
```bash
ip ospf dead-interval <№>  # В секундах (по умолчанию 40)
ip ospf hello-interval <№>  # В секундах (1-65535)
```

Создание ключа аутентификации OSPF:
```bash
ip ospf authentication-key <Ключ>
ip ospf message-digest-key <id> md5 <Ключ>
```

---
### Создание/редактирование интерфейса:

```bash
interface <имя интерфейса>
	ip address <IP-адрес>/<Префикс>
	description "Описание"  # Текст в кавычках
```

Включение порта:
```bash
port <порт>
service-instance <имя>
	encapsulation untagged
	connect ip interface <имя интерфейса>
```

---
### Определение пула NAT:

```bash
ip nat pool <имя_пула> <локальный_IP1> <локальный_IP2>
```

Настройка NAT на интерфейсе:
```bash
interface <имя интерфейса>
	ip nat inside  # Внутренний интерфейс
	ip nat outside # Внешний интерфейс
```

```bash
ip nat source dynamic inside-to-outside pool <имя_пула> overload interface <внешний_интерфейс>
```

Проверка трансляции NAT:
```bash
$ show ip nat translation
```
---
### Настройка DHCP
```tex
> enable
$ conf t
(config)# ip pool <НАЗВАНИЕ ПУЛА> 192.168.0.1-192.168.0.255 # пул можно указать другой, это ПРИМЕР
(config)# dhcp-server 1 или любая другая цифра
(config-dhcp-server)# lease 3600
(config-dhcp-server)# dns <DNS-Сервер-IP>
(config-dhcp-server)# domain-name au-team.irpo
(config-dhcp-server)# pool <НАЗВАНИЕ-ПУЛА> 10
(config-dhcp-server-pool)# mask 255.255.255.240 или mask 28
(config-dhcp-server-pool)# gateway <ШЛЮЗ-IP>
(config-dhcp-server-pool)# exit
(config-dhcp-server)# static ip <Адрес из пула, будет назначен статически клиенту>
(config-dhcp-server-static)# chaddr <МАК-АДРЕС-КЛИЕНТА> # Кому будет присваивать статический IP
(config-dhcp-server-static)# mask 255.255.255.240 или mask 28 
(config-dhcp-server-static)# gateway <ШЛЮЗ-IP>
(config-dhcp-server-static)# dns <DNS-Сервер-IP>
(config-dhcp-server-static)# exit
(config-dhcp-server)# exit
(config)# interface <INT> # Название интерфейса, идущего к клиенту/клиентам
(config-if)# dhcp-server 1 # цифра, которую вы назначали при создании dhcp-server
(config-if)# exit
```
---
