— Перейти в [[HUB]] —
> [!INFO] **Режимы работы устройства**
> - **enable** — Привилегированный режим `{en}`
> - **configure terminal** — Режим конфигурации `{conf t}`

> [!INFO] Содержимое
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

---
## Базовая настройка устройства

```bash
hostname [Имя устройства]  
# Устанавливает имя устройства.
```

```bash
enable secret P@ssw0rd  
# Устанавливает защищённый пароль для привилегированного режима.
```

```bash
line vty 0 4-15  
password P@ssw0rd  
login  
exit  
# Настройка пароля для удалённого доступа (Telnet/SSH).
```

```bash
line console 0  
password P@ssw0rd  
login  
exit  
# Настройка пароля для консольного подключения.
```

```bash
service password encryption  
# Включает шифрование всех паролей в конфигурации.
```

```bash
banner motd 'Введите текст'  
# Устанавливает текст приветственного баннера (Message of the Day).
```

```bash
show running config  
# Просмотр текущей конфигурации устройства.
```
---
### Очистка и сохранение конфигурации
```bash
erase startup config  
# Удаляет сохранённую конфигурацию.
```

```bash
copy running-config startup-config  
write  
wr  
# Сохраняет текущую конфигурацию в память.
```
---
### Настройка VLAN
```bash
interface vlan 1  
ip address 192.168.100.10 255.255.255.0  
no shutdown  
# Назначает IP-адрес для VLAN 1 и активирует её.
```

```bash
show ip interface  
# Показывает состояние интерфейсов и их IP-адреса.
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
# Создаёт VLAN, назначает их на порты и настраивает trunk-порт.

```
---
### Spanning Tree (STP)
```bash
spanning-tree vlan VLAN-ID root primary  
# Настраивает приоритет STP для указанного VLAN как primary.
```

```bash
spanning-tree vlan VLAN-ID priority value  
# Задаёт приоритет для указанного VLAN.  
# Пример:
spanning-tree vlan 1 priority 24576  
# Приоритет в диапазоне от 0 до 61440.
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
# Включает PortFast и защиту BPDU на порту.
```
Проверка:
```bash
show spanning tree  
# Показывает состояние Spanning Tree.
```
---
### DHCP
```bash
service dhcp  
ip dhcp pool VLAN10  
network 192.168.10.0 255.255.255.0  
default-router 192.168.10.1  
dns-server 8.8.8.8  

ip dhcp excluded-address 192.168.10.1  
# Настраивает DHCP-сервер с исключёнными адресами.
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
# Настраивает NAT (перегрузка).
```
Проверка NAT
```bash
show ip nat translations  
show ip nat statistics
```
---
### Настройка SSH
```bash
ip domain-name mydomain.local  
# Устанавливает доменное имя устройства.

crypto key generate rsa  
# Генерирует RSA-ключ для SSH (рекомендуется ключ длиной 2048).  
# При запросе длины ключа укажите: 2048.

ip ssh version 2  
# Включает использование SSH версии 2.

line vty 0 4  
transport input ssh  
login local  
exit  

username admin privilege 15 password P@ssw0rd  
# Создаёт пользователя `admin` с максимальным уровнем доступа.
```
---
### Настройка NTP(синхронизация времени)
```bash
ntp server 192.168.1.1  
# Указывает NTP-сервер для синхронизации времени.

clock timezone UTC +3  
# Устанавливает часовой пояс.

show ntp status  
# Проверка синхронизации с NTP-сервером.
```
---
### Настройка SNMP (Мониторинг устройства)
```bash
snmp-server community public RO  
# Устанавливает строку сообщества для чтения (`public`).

snmp-server location ServerRoom  
snmp-server contact admin@mydomain.local  
# Добавляет описание местоположения и контакт администратора.

snmp-server enable traps  
# Включает отправку SNMP-уведомлений.
```
---
### Настройка Port Security(Безопасность портов)
```bash
interface FastEthernet0/1  
switchport port-security  
switchport port-security maximum 2  
# Ограничивает число устройств, подключаемых к порту (до 2).  

switchport port-security mac-address sticky  
# Позволяет автоматически сохранять MAC-адреса подключённых устройств.

switchport port-security violation shutdown  
# Отключает порт при нарушении правил.

show port-security  
# Проверяет статус Port Security.
```
---
### Проверка и отладка
```bash
show version  
# Отображает информацию об устройстве и его работе.

show ip interface brief  
# Показывает состояние интерфейсов и IP-адреса.

show logging  
# Показывает логи системы.

debug ip nat  
# Включает отладку NAT (осторожно, может замедлить работу устройства).

undebug all  
# Отключает отладку.
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
# Создаёт туннель, задаёт IP-адрес, исходный и целевой IP для туннеля.
```
Устройство Б
```bash
interface Tunnel0  
ip address 192.168.1.2 255.255.255.252  
tunnel source 198.51.100.1  
tunnel destination 203.0.113.1  
no shutdown  
# Создаёт туннель на удалённом конце.
```
Проверка состояния туннеля
```bash
show ip interface brief | include Tunnel  
# Показывает состояние туннельного интерфейса.

show run interface Tunnel0  
# Проверяет настройки туннеля.

ping 192.168.1.2 source 192.168.1.1  
# Тестирует туннель.
```
---
### Настройка NetFlow (Анализ трафика)
```bash
ip flow-export destination 192.168.1.100 9996  
ip flow-export version 9  
ip flow-export source FastEthernet0/0  
# Настраивает экспорт NetFlow на сервер.

interface FastEthernet0/0  
ip flow ingress  
ip flow egress  
# Включает сбор данных NetFlow на интерфейсе.

show ip cache flow  
# Показывает текущие данные NetFlow.
```
---

```bash

```

```bash

```
---
