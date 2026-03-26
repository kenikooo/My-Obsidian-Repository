— Перейти в [[HUB]] —
> [!INFO] **Режимы работы устройства**
> - **enable** — Привилегированный режим `{en}`
> - **configure terminal** — Режим конфигурации `{conf t}`

> [!INFO] Содержание
1. [[#Базовая настройка устройства]]
2. [[#Очистка и сохранение конфигурации]]
3. [[#Настройка VLAN]]
4. [[#Spanning Tree (STP)]]
5. [[#DHCP]]
6. [[#NAT]]
7. [[#Настройка SSH]]
8. [[#Настройка NTP(синхронизация времени)]]
9. [[#Настройка SNMP (Мониторинг устройства)]]
10. [[#Настройка Port Security(Безопасность портов)]]
11. [[#Проверка и отладка]]
12. [[#Настройка GRE-Туннеля]]
13. [[#Настройка NetFlow (Анализ трафика)]]
14. [[#Настройка IP-Телефонии]]
15. [[#Настройка Dial-Plan]]

---
## Базовая настройка устройства

```bash
hostname [Имя устройства]  
# Устанавливает имя устройства
```

```bash
enable secret P@ssw0rd  
# Устанавливает защищенный пароль для привилегированного режима
```

```bash
line vty 0 4-15  
password P@ssw0rd  
login  
exit  
# Настройка пароля для удаленного доступа (Telnet/SSH)
```

```bash
line console 0  
password P@ssw0rd  
login  
exit  
# Настройка пароля для консольного подключения
```

```bash
service password encryption  
# Включаем шифрование всех паролей в конфигурации
```

```bash
banner motd 'Введите текст'  
# Устанавливаем текст приветственного баннера
```

```bash
show running config  
# Вывод текущей конфигурации устройства
```
---
### Очистка и сохранение конфигурации
```bash
erase startup config  
# Удаляем сохраненную конфигурацию
```

```bash
copy running-config startup-config  
write  
wr  
# Сохраняем текущую конфигурацию в память
```
---
### Настройка VLAN
```bash
interface vlan 1  
ip address 192.168.100.10 255.255.255.0  
no shutdown  
# Назначаем IP-адрес для VLAN 1 и активируем ее
```

```bash
show ip interface  
# Показываем состояние интерфейсов и их IP-адреса
```

```bash
vlan 10  
name Management  

vlan 20  
name Sales  

vlan 30  
name HR  

interface FastEthernet0/1  
switchport mode access  
switchport access vlan 10  
no shutdown  

interface FastEthernet0/24  
switchport mode trunk  
switchport trunk allowed vlan 10,20,30  
no shutdown  
# Создаем VLAN, назначаем их на порты и настраиваем trunk-порт

```
---
### Spanning Tree (STP)
```bash
spanning-tree vlan VLAN-ID root primary  
# Настраиваем приоритет STP для указанного VLAN как primary
```

```bash
spanning-tree vlan VLAN-ID priority value  
# Задаем приоритет для указанного VLAN
# Пример:
spanning-tree vlan 1 priority 24576  
# Приоритет в диапазоне от 0 до 61440
```
Пример использования:
```bash
spanning-tree VLAN1 priority 24576  
end  

spanning-tree VLAN1 root secondary  
end  
```
На интерфейсах:
```bash
interface FastEthernet0/11  
spanning-tree portfast  
spanning-tree bpguard enable  
end  
# Включаем PortFast и защиту BPDU на порту
```
Проверка:
```bash
show spanning tree  
# Вывод состояния Spanning Tree
```
---
### DHCP
```bash
service dhcp  
ip dhcp pool <Имя пула>  
network 192.168.10.0 255.255.255.0  
default-router 192.168.10.1  
dns-server 8.8.8.8  

ip dhcp excluded-address 192.168.10.1  
# Настраиваем DHCP-сервер с исключенными адресами
```
---
### NAT
```bash
interface FastEthernet0/0  
ip address 203.0.113.1 255.255.255.248  
ip nat outside  
no shutdown  

interface FastEthernet0/1  
ip address 192.168.10.1 255.255.255.0  
ip nat inside  
no shutdown  

access-list 1 permit 192.168.0.0 0.0.255.255  

ip nat inside source list 1 interface FastEthernet0/0 overload  
# Настраиваем NAT (перегрузка)
```
Проверка NAT
```bash
show ip nat translations  
show ip nat statistics
```
#### Более подробно
Размечаем интерфейсы
```bash
interface gi1 # Настраиваем внешний интерфейс
ip nat outside
interface gi2 # Настраиваем внутренний интерфейс
ip nat inside
```
Настраиваем трансляцию сетей (ACL)
```bash
access-list 1 permit 192.168.100.0 0.0.0.255
```
- Цифра 1 - номер списка (может быть любым)
Включаем NAT
```bash
ip nat inside source list 1 interface gi1 overload
```
Расшифровка:
- `inside source` - транслируем изнутри наружу
- `list 1` - используем ACL номер 1
- `interface gi2` - через внешний интерфейс
- `overload` - PAT (много клиентов - один IP)
---
### Настройка SSH
```bash
ip domain-name mydomain.local  
# Устанавливаем доменное имя устройства

crypto key generate rsa  
# Генерируем RSA-ключ для SSH (рекомендуется ключ длиной 2048). 
# При запросе длины ключа указываем: 2048

ip ssh version 2  
# Включаем использование SSH версии 2

line vty 0 4  
transport input ssh  
login local  
exit  

username admin privilege 15 password P@ssw0rd  
# Создаем пользователя `admin` с максимальным уровнем доступа
```
---
### Настройка NTP(синхронизация времени)
```bash
ntp server 192.168.1.1  
# Указываем NTP-сервер для синхронизации времени

clock timezone UTC +3  
# Устанавливаем часовой пояс

show ntp status  
# Проверяем синхронизацию с NTP-сервером
```
---
### Настройка SNMP (Мониторинг устройства)
```bash
snmp-server community public RO  
# Устанавливаем строку сообщества для чтения (`public`)

snmp-server location ServerRoom  
snmp-server contact admin@mydomain.local  
# Добавляем описание местоположения и контакт администратора

snmp-server enable traps  
# Включаем отправку SNMP-уведомлений
```
---
### Настройка Port Security(Безопасность портов)
```bash
interface FastEthernet0/1  
switchport port-security  
switchport port-security maximum 2  
# Ограничиваем число устройств, подключаемых к порту (до 2)

switchport port-security mac-address sticky  
# Позволяем автоматически сохранять MAC-адреса подключённых устройств

switchport port-security violation shutdown  
# Отключаем порт при нарушении правил

show port-security  
# Проверяем статус Port Security
```
---
### Проверка и отладка
```bash
show version  
# Вывод информации об устройстве и его работе

show ip interface brief  
# Вывод состояния интерфейсов и IP-адреса

show logging  
# Вывод логов системы

debug ip nat  
# Включаем отладку NAT (осторожно, может замедлить работу устройства)

undebug all  
# Отключаем отладку
```
---
### Настройка GRE-Туннеля
> **GRE (Generic Routing Encapsulation)** используется для создания туннелей между удалёнными сетями. Это полезно для маршрутизации трафика через несоприкасающиеся сети.

Устройство А
```bash
interface Tunnel0  
ip address 192.168.1.1 255.255.255.252  
tunnel source 203.0.113.1  
tunnel destination 198.51.100.1  
no shutdown  
# Создаем туннель, задаем ему IP-адрес, исходный и целевой IP для туннеля
```
Устройство Б
```bash
interface Tunnel0  
ip address 192.168.1.2 255.255.255.252  
tunnel source 198.51.100.1  
tunnel destination 203.0.113.1  
no shutdown  
# Создаем туннель на удаленном конце
```
Проверка состояния туннеля
```bash
show ip interface brief | include Tunnel  
# Выводим состояние туннельного интерфейса

show run interface Tunnel0  
# Проверяем настройки туннеля

ping 192.168.1.2 source 192.168.1.1  
# Тестируем туннель
```
---
### Настройка NetFlow (Анализ трафика)
```bash
ip flow-export destination 192.168.1.100 9996  
ip flow-export version 9  
ip flow-export source FastEthernet0/0  
# Настраиваем экспорт NetFlow на сервер

interface FastEthernet0/0  
ip flow ingress  
ip flow egress  
# Включаем сбор данных NetFlow на интерфейсе

show ip cache flow  
# Вывод текущих данных NetFlow
```
---
### Настройка IP-Телефонии
Активируем DHCP
```bash
service dhcp
```
Вешаем IP-Адрес на интерфейс
```bash
interface <имя>
ip address <RTR-IP> <SUBNET>
no shutdown
```
Исключаем наш адрес из выдачи DHCP
```bash
ip dhcp excluded-address <RTR-IP>
```
Создаем пул
```bash
ip dhcp pool phones
	network 192.168.10.0 255.255.255.0
	default-router <RTR-IP>
	option 150 ip <RTR-IP>
```
Активируем функции IP-Телефонии
```bash
telephony-service
max-ephones 10 # Макс. кол-во поддерживаемых телефонов
max-dn 10 # Макс. кол-во поддерживаемых номеров
ip source-address <RTR-IP> port 2000
auto assign 1 to 10
```
Задаем номер
```bash
ephone-dn 1
number 1668
```

> Все что ниже делается в `DHCP (Option 150)`

Разрешаем звонок
```bash
voice service voip
 allow-connections sip to sip
 sip
  registrar server expires max 3600 min 3600
 exit
```
Активируем набор codec'ов
```bash
voice class codec 1 # Объединяет кодеки
	codec preference 1 g711alaw # g711alaw самый распространенный
	codec preference 2 g711ulaw # Для Европеской (Америнканские стандарты) связи
	codec preference 3 g729br8
	
```
Настройка телефонной станции
```bash
voice register global
	mode cme
	source-address <RTR-IP> port 5060 # Порт для SIP
	max-dn 50
	max-pool 50
```
Регистрация телефонов и софт-фонов
```bash
voice register dn 1
	number <Придумайте-Номер>
```
Регистрируем пул
```bash
voice register pool 1
	id mac AAAA.DDDD.CCCC # Формат Cisco
# Привязываем номер к пулу (указывается любой)
	number 1 dn 1
# Привязываем набор codec
	voice class codec 1
# Указать имя пользователя и пароль
	username test password Passw0rd # Вместо test указывается пользователь, который будет звонить. Именно пароль Passw0rd проходит (не через @)
```

>[!INFO] Для Windows
> [`MicroSIP`](https://help.mobilon.ru/equipment/softphones/microsip/)

> [!WARNING] Для Linux
> [`LinPhone`](https://redos.red-soft.ru/base/redos-7_3/7_3-users-tasks/7_3-chat/7_3-linphone/)

---
### Настройка Dial-Plan
`dial-plan` на пропуск всего
```bash
dial-peer voice 10 voip
session protocol sipv2
incoming called-number .
```
`dial-plan` на удаленную рабочую станцию
```bash
dial-peer voice 200 voip
destination-pattern 2.. 
session protocol sipv2 
session target ipv4:192.168.100.2 
voice-class codec 1 
dtmf-relay rtp-nte 
```
`dial-plan` на внутренние номера
```bash
destination-pattern 1
session protocol sipv2
session target sip-server
voice-class codec 1
dtmf-relay rtp-nte
```

---