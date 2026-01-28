— Перейти в [[HUB]] —
Содержание:
1. [[#Первоначальная установка]]
2. [[#Добавление интерфейсов]]
3. [[#Разрешения/ограничения по портам]]

### Первоначальная установка
При установке, он запускается в live-cd
Для перехода к полноценной установке, используем следующие данные для входа:
```bash
> login: installer
> password: opnsense
```
Для входа после установки (или просто в live-cd):
```bash
> login: root
> password: opnsense # (или свой, который установлен при установке)
```
Для подключения интернета к маршрутизатору, осуществляем вход и выбираем '1', далее, отвечаем на вопросы:
```bash
> Do you want to configure LAGGs now? [y/N]: n
> Do you want to configure VLAN now? [y/N]: n

> Enter the WAN interface name or 'a' for auto-detection: em0 # (интерфейс, который идет в Интернет)
> Enter the LAN interface name or 'a' for auto-detection: em1 # (интерфейс, который идет в внутреннюю сеть)
> На следующем вопросы ничего не указываем, просто Enter
```
### Добавление интерфейсов
```bash
Interfaces -> Assignments
Следуем шагам, описанным там (если есть доступные девайсы - интерфейсы), Вы сможете их добавить и поставить на них Enable Interface
```
### Разрешения/ограничения по портам
Создадим группу портов:
```bash
Firewall -> Aliases
Жмем "+" и добавляем правило:
> Name: AllowedPort
> Type: Port(s)
> Content: 53, 67, 68, 80, 88, 123, 135, 139, 443, 445, 636
[SAVE]
```
Создаем группу интерфейсов:
```bash
Firewall -> Groups
Жмем "+" и добавляем правило:
> Name: Internal_NET
> Members: Выбираем всех участников локальной сети, у меня это: MGMT, SRV, CLI
[SAVE]
```
Следующим шагом, настраиваем правила:
```bash
Firewall -> Rules -> Internal_NET (или как Вы назвали группу интерфейсов)
Жмем "+" и добавляем правило:
> Action: Pass
> Interface: Internal_NET
> Protocol: TCP/UDP
> Source: any
> Destination: any
> Destination port range:
Добавляем наше правило: from AllowedPort to AllowedPort
[SAVE]
```
Теперь, создаем запрещающее правило, принцип остается тем же:
```bash
Firewall -> Rules -> Internal_NET (или как Вы назвали группу интерфейсов)
Жмем "+" и добавляем правило:
> Action: Block # Вместо Pass - Block
> Interface: Internal_NET
> Protocol: TCP/UDP
> Source: any
> Destination: any
[SAVE]
```
Если в офисе требуется пинг (ICMP-запросы), создаем еще одно правило:
```bash
Firewall -> Rules -> Internal_NET (или как Вы назвали группу интерфейсов)
Жмем "+" и добавляем правило:
> Action: Pass
> Interface: Internal_NET
> Protocol: ICMP
> ICMP Type: Echo Request, Echo Reply
> Source: any
> Destination: any
[SAVE]
```