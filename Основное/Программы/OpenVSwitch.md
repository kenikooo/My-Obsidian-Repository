— Перейти в [[HUB]] —
> [!INFO] Что такое **OpenVSwitch**?
Программный многоуровневый коммутатор с открытым исходным кодом, предназначенный для работы в средах виртуализации и аппаратной виртуализации.

---
> [!tldr] Используемые дистрибутивы
> 
> - openvswitch

---

- Для работы все используемые физические интерфейсы должны быть включены.

> [!WARNING] Включаем сервис openvswitch и добавляем в автозапуск
> 
> ```bash
> systemctl enable --now openvswitch.service  
> ```

- Создаём папку моста `/etc/net/ifaces/ovs0` и папку виртуального интерфейса `/etc/net/ifaces/mgmt` (имена могут быть любыми).

> [!code] Создание директорий
> 
> ```bash
> mkdir /etc/net/ifaces/ovs0  
> mkdir /etc/net/ifaces/mgmt  
> ```

- Создаём файл `options` в каждой папке и редактируем.

> [!tip] Файл конфигурации `/etc/net/ifaces/ovs0/options`
> 
> ```ini
> TYPE=ovsbr # Тип интерфейса — bridge  
> HOST='ens33 ens34 ens36' # Указываем порты  
> ```

> [!info] Файл конфигурации `/etc/net/ifaces/mgmt/options`
> 
> ```ini
> TYPE=ovsport # Тип интерфейса — port  
> BOOTPROTO=static # Тип адреса (static | dhcp)  
> BRIDGE=ovs0 # Мост, к которому привяжем (существующий)  
> VID=300 # Vlan ID  
> ```

- Создаём и заполняем `/etc/net/ifaces/mgmt/ipv4address` и `/etc/net/ifaces/mgmt/ipv4route` для назначения адреса на интерфейс `mgmt`.
    
- Перезагружаем сервис `network`.
    

> [!code] Перезапуск сервиса
> 
> ```bash
> systemctl restart network  
> ```

- Чтобы настройки сохранились после перезагрузки, изменяем параметр в `/etc/net/ifaces/default/options`.

> [!code] Отключение удаления настроек
> 
> ```ini
> OVS_REMOVE=no # Выключаем удаление настроек  
> ```

- Для работы необходимо включить модуль ядра `8021q`.

> [!code] Включение модуля
> 
> ```bash
> modprobe 8021q # Временно включаем модуль  
> echo "8021q" | tee -a /etc/modules # Постоянное включение модуля  
> ```

> [!info] Производим настройку интерфейсов
> 
> ```bash
> ovs-vsctl set port ens33 trunk=100,200 # Настроить trunk на VLAN'ы. Если не указывать VLAN, то пропустит всё  
> ovs-vsctl set port ens34 tag=100 # Настроить access на VLAN  
> ovs-vsctl set port ens35 tag=200  
> ovs-vsctl show # Вывод настроек  
> ```
