— Перейти в [[HUB]] —
> [!INFO ] Что такое **DHCP-Server**? 
> Cетевой сервер, который автоматически назначает устройствам в локальной сети IP-адреса и другие сетевые параметры, такие как маска подсети, шлюз по  умолчанию и адреса DNS-серверов. 

Установка дистрибутивов:
```bash
apt-get update && apt-get install dhcp-server -y 
```
Переходим в конфиг файл:
```bash
vim /etc/sysconfig/dhcpd
# В файле, мы дополняем одну строчку:
DHCPDARGS=enp3s0 # ИЛИ ДРУГОЙ ИНТЕРФЕЙС ИДУЩИЙ НА ВЫХОД
```
Далее - конфигурация DHCP-сервера:
```bash
vim /etc/dhcp/dhcpd.conf
# Можем очистить весь конфиг или начать с начала (если конфига нет, создаем):
option domain-name "au-team.irpo";
option domain-name-servers 192.168.0.10; # Адрес HQ-SRV
authoritative;

subnet 192.168.0.32 netmask 255.255.255.240 {
	range 192.168.0.34 192.168.0.46;
	option routers 192.168.0.33;
	option domain-name "au-team.irpo";
	default-lease-time 600;
	max-lease-time 7200;
}
```
Меняем интерфейс (на котором будет происходить раздача):
```bash
vim /etc/sysconfig/dhcpd
> DHCPDARGS="подинтерфейс/интерфейс"
# Пример на подинтерфейсе (для интерфейса аналогично)
> DHCPDARGS="enp0s3.100" 
```
Запуск и проверка службы:
```bash
systemctl enable --now dhcpd
systemctl restart dhcpd
systemctl status dhcpd # Если там есть ошибки, смотрите что не так, возможно допущены ошибки во время написания конфига
```
На HQ-CLI получаем адрес по DHCP:
```bash
# Ставим получение адреса по DHCP через конфиг или же графику
ip -c a
# В терминале (если адрес НЕ прилетел): dhcpd
```
