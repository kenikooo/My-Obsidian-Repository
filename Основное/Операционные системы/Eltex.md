— Перейти в [[HUB]] —
Содержание:
1. [[#Общие команды]]
2. [[#Команды просмотра]]
3. [[#Установка баннера]]
4. [[#Создание пользователя]]
5. [[#Настройка IP Адресации]]
6. [[#Настройка IPv6]]
7. [[#Настройка шлюза]]
8. [[#Настройка NAT]]
9. [[#Настройка GRE-туннеля]]
10. [[#Настройка Netflow]]
### Общие команды  
```bash
commit # Применить изменения  
confirm # Подтвердить изменения  
restore # Отмена неподтвержденных изменений  
rollback # Откат подтвержденных изменений  
config # Вход в режим конфигурации (НЕ conf t, как в Cisco)
````

---
### Команды просмотра
```bash
show running-configure # Просмотр общих настроек  
show ip interfaces # Просмотр IP-адресов на интерфейсах
show interfaces status # Просмотр интерфейсов  
show netflow statistics # Просмотр статистики netflow 
````

---
### Установка баннера

```bash
banner login "Unauthorized connection is prosecuted by the law of the Russian Federation" # Баннер до аутентификации пользователя  
banner exec "Hello! This is a vESR!" # Баннер после аутентификации  
```

---
### Создание пользователя

```bash
username <name> # Создание пользователя  
	password <pas>  # Задание пароля  
	privilege <№_1-15> # Уровень привилегий (10-14 — настройка системы, но не полный доступ)  
exit # Выход  
```

---
### Настройка IP Адресации

```bash
interface <int_name> # Переход в режим интерфейса  
	description "<описание>" # Описание интерфейса (до 31 символа)  
	ip firewall disable # Отключение firewall  
	ip address <ip>/<prefix> # Задание IP-адреса (пример: 192.168.0.0/32)
	ip netflow export # Включаем сбор экспорта статистики netflow
```

---
### Настройка IPv6

```bash
ipv6 enable # Включение IPv6 на устройстве  
ipv6 address <ipv6>/<prefix> # Назначение IPv6-адреса на интерфейс  
```

---
### Настройка шлюза

```bash
ip route <сеть>/<префикс> <ip_next_hop> # Добавление маршрута IPv4  
ipv6 route <префикс>/<сеть> <ipv6_next_hop> # Добавление маршрута IPv6  
```

---
### Настройка NAT

```bash
# Создаем группу объектов для локальных сетей
object-group network LOCAL_NET
	ip address-range 192.168.0.1-192.168.0.254
	ip address-range 192.168.1.1-192.168.1.254
exit

# Конфиг пул для SNAT с внешним IP
nat source
	pool WAN_POOL
		ip address-range 10.0.2.15
	exit
exit

# Создаем правила SNAT
nat source
	ruleset SNAT
		to interface gi1/0/2
		rule 1
			match source-address LOCAL_NET
			action source-nat pool WAN_POOL
			enable
		exit
	exit
exit

# Сохраняем конфиг
# В привел. режиме
commit
confirm
```

---
### Настройка GRE-туннеля

```bash
interface Tunnel0 # Создание туннельного интерфейса 
	ip address <ip>/<prefix> # Назначение IP-адреса 
	tunnel source <source_ip> # Указание исходного IP
	tunnel destination <dest_ip> # Указание целевого IP
exit
commit # Сохранение изменений  
confirm # Подтверждение сохранения  
```

---
#### Настройка Netflow

```bash
netflow collector [адрес] # Указываем IP-Адрес коллектора
int gi1/0/1 # Включаем сбор экспорта статистики netflow
	ip netflow export
netflow enable # Активируем Netflow на Маршрутизаторе
```

---

