— Перейти в [[HUB]] —
> [!INFO] Что такое **Firewalld**?
Программа управления межсетевым экраном для ОС Linux, которая контролирует и фильтрует сетевой трафик, защищая сеть от несанкционированного доступа.

> [!WARNING] Существующие дистрибутивы
 `firewalld` 
 `firewall-config`

### Общие команды
```bash
systemctl enable --now firewalld # Запустить службу и поместить в автозагрузку
systemctl disable firewalld # Отключить службу
systemctl start firewalld # Принудительный старт службы
systemctl stop firewalld # Принудительная остановка службы
systemctl status firewalld # Узнать статус службы

firewall-cmd --state # Статус службы
firewall-cmd --reload # Перезапуск зон
firewall-cmd --complete-reload # Перезагрузить конфигурацию мгновенно применив все изменения
```
### Флаги для работы с зонами
```bash
firewall-cmd --list-all-zones # Выводит список всех зон
firewall-cmd --get-active-zones # Выводит список активных зон
firewall-cmd --get-zones # Выводит все доступные зоны
> Информация о конкретной зоне:
firewall-cmd --zone=public --list-all # Внутренние интерфейсы
firewall-cmd --zone=internal --list all # Внешние интерфейс
```