— Перейти в [[HUB]] —
> [!VOTE] Что такое **FreeIPA**?
Открытое программное обеспечение для Linux, обеспечивающее централизованное управление учетными записями, аутентификацией (Kerberos) и политиками доступа (LDAP)

> [!INFO] Содержимое
1. [[#Настройка FreeIPA без интегрированного DNS]]
2. [[#Установка и настройка FreeIPA с интегрированным DNS]]
### Настройка FreeIPA без интегрированного DNS
Для корректной работы сервера, задаем ему полное доменное имя в формате `name.domain.local`
Отключаем работающие на порту 8080 ahttpd во избежание конфликтов с разворачиваемым tomcat и отключить HTTP в Apache2:
```bash
service ahttpd stop
a2dissite 000-default_https
a2disport https
service http2 conderload
```
Для ускорения установки, ставим демон энтропии haveged:
```bash
apt-get update && apt-get install haveged
```
После чего, включаем его:
```bash
systemctl enable --now haveged
```
Приступим к установке FreeIPA-Server, без интегрированного DNS
```bash
apt-get update && apt-get install freeipa-server
```
Для удобства, запустим интерактивную установку, выполнив команду:
```bash
ipa-server-install
```
На первый вопрос, нужно ли сконфигурировать DNS-сервер BIND, отвечаем `yes`. Далее указываем имя узла, на котором будет установлен сервер FreeIPA, доменное имя и пространство Kerberos.

Используемая документация:
> [Установка сервера FreeIPA](https://www.altlinux.org/FreeIPA/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0_FreeIPA)

---
### Установка и настройка FreeIPA с интегрированным DNS

В самом начале производим установку необходимого дистрибутива:
```bash
apt-get update && apt-get install haveged tzdata
```
Запускаем:
```bash
systemctl enable --now haveged
```
Настроим корректный часовой пояс:
```bash
timedatectl set-timezone Europe/Moscow
```
Проведем установку FreeIPA:
```bash
apt-get update && apt-get install freeipa-server-dns
```
Команда установки сервера:
```bash
ipa-server-install -U --hostname=$(hostname) \
-r AU-TEAM.CLOUD -n au-team.cloud -p P@ssw0rd -a P@ssw0rd \
--setup-dns --forwarder 8.8.8.8 --auto-reverse
```
Получаем билет Kerberos:
```bash
kinit admin
# Пароль, который был задан в команде выше, под флагом -a
```
Документация, из которой взята настройка:
> [ALT Linux | Глава 53. FreeIPA](https://docs.altlinux.org/ru-RU/alt-server/11.0/html/alt-server/ch53.html)

На клиенте, под управлением ALT Linux, произведем установку дистрибутива для присоединения к домену FreeIPA:
```bash
apt-get update && apt-get install task-auth-freeipa -y
```
На клиенте, домен поиска: `au-team.cloud`, а в качестве dns-сервера, опять же наш сервер: `192.168.0.101` 
> [!WARNING] IP-Адрес и Домен будут отличаться, все проведенные настройки были выполнены в учебных целях

### Удаление домена
Обычно требуется если установка не завершилась корректно или завершилась, но частично.
>[!BUG] **ПОЛНОСТЬЮ СТИРАЕТ ДОМЕН**
>`ipa-server-install --uninstall -U`
