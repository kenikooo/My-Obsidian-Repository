— Перейти в [[HUB]] —
> [!INFO] Что такое **Bind**?
Программное обеспечение для DNS-сервера (системы доменных имен)

```bash
mkdir -p /etc/bind/zone
vim /etc/bind/zone/db.au-team.irpo
```
Содержимое /etc/bind/zone/db.au-team.irpo:
```bash
$TTL    86400 ;
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                          2024120101 ; Serial
                          3600       ; Refresh
                          1800       ; Retry
                          604800     ; Expire
                          86400 )    ; MinimumTTL

@       IN      NS      hq-srv.au-team.irpo.

hq-srv  IN      A       192.168.0.10
hq-rtr  IN      A       192.168.0.1
        IN      A       192.168.0.33

hq-cli  IN      A       192.168.0.40
br-srv  IN      A       192.168.1.10
br-rtr  IN      A       192.168.1.1

hq-tun  IN      A       10.0.0.1
br-tun  IN      A       10.0.0.2
```

Переходим к следующей зоне:
```bash
vim /etc/bind/zone/db.192.168.0
```
Содержимое:
```bash
$TTL    86400
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                          2024120101 ; Serial
                          3600       ; Refresh
                          1800       ; Retry
                          604800     ; Expire
                          86400 )    ; Minimum TTL

@       IN      NS      hq-srv.au-team.irpo.

10      IN      PTR     hq-srv.au-team.irpo.
1       IN      PTR     hq-rtr.au-team.irpo.
33      IN      PTR     hq-rtr.au-team.irpo.
40      IN      PTR     hq-cli.au-team.irpo.
```
Переходим к следующей зоне:
```bash
vim /etc/bind/zone/db.192.168.1
```
Содержимое:
```bash
$TTL    86400
@       IN      SOA     au-team.irpo. root.au-team.irpo. (
                          2024120101 ; Serial
                          3600       ; Refresh
                          1800       ; Retry
                          604800     ; Expire
                          86400 )    ; Minimum TTL

@       IN      NS      hq-srv.au-team.irpo.
10      IN      PTR     br-srv.au-team.irpo.
1       IN      PTR     br-rtr.au-team.irpo.
```
Конфигурируем основные файлы bind'а:
```bash
vim /etc/bind/local.conf
```
Содержимое /etc/bind/local.conf: 
```bash
zone "au-team.irpo" {
    type master;
    file "/etc/bind/zone/db.au-team.irpo";
    allow-query { any; };
};

zone "0.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zone/db.192.168.0";
    allow-query { any; };
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zone/db.192.168.1";
    allow-query { any; };
};
```
Настраиваем DNS-пересылки:
```bash
vim /etc/bind/options.conf
```
Содержимое (просто подменяем/дополняем и расскоментируем нужное) /etc/bind/options.conf:
```bash
options {
    directory "/etc/bind/zone"; # Неизменна 

    recursion yes; # Дописываем
    allow-recursion { any; }; # Дописываем
    
    forwarders { # Раскоментим и дописываем
        77.88.8.7;
        77.88.8.3;
        8.8.8.8;
    };
    forward only; # Расскоментим
    
    listen-on { any; }; # Заменить адрес на ANY
    allow-query { 192.168.0.0/24; 192.168.1.0/24; }; # Заменить запись на адреса или any (для всех типа)
};
```
Проверка синтаксиса:
```bash
named-checkconf
# Перезапустим bind:
systemctl enable --now bind
systemctl restart bind
systemctl status bind # Если ошибок нет - все в норме
```
Настройка разрешения имен на самом сервере HQ-SRV:
```bash
vim /etc/resolv.conf
# Добавляем следующую строку:
nameserver 127.0.0.1
```
Проверка работоспособности:
```bash
nslookup hq-cli.au.team.irpo 127.0.0.1
host 192.168.1.10
# и т.д
```
На клиентах,  дополнение в resolv.conf будет выглядеть следующим образом:
```bash
nameserver 192.168.0.10
search au-team.irpo
```
Используемая документация: 
- 📖 [bind](https://docs.altlinux.org/ru-RU/domain/10.4/html/alt-domain-p10/bind-options.html)
- 📰 [Настройка DNS-сервера bind RedOS конфиг файлы](https://redos.red-soft.ru/base/redos-8_0/8_0-network/8_0-customize-dns/8_0-customize-dns-bind/)