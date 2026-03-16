— Перейти в [[HUB]] —
> [!INFO] Что такое **FreeIPA**?
Открытое программное обеспечение для Linux, обеспечивающее централизованное управление учетными записями, аутентификацией (Kerberos) и политиками доступа (LDAP)

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
Приступим к установке `FreeIPA-Server` с интегрированным DNS:
```bash
apt-get update && apt-get install freeipa-server-dns
```
Если же DNS не требуется, устанавливаем следующий пакет:
```bash
apt-get update && apt-get install freeipa-server
```
Для удобства, запустим интерактивную установку, выполнив команду:
```bash
ipa-server-install
```
На первый вопрос, нужно ли сконфигурировать DNS-сервер BIND, отвечаем `yes`. Далее указываем имя узла, на котором будет установлен сервер FreeIPA, доменное имя и пространство Kerberos.

#### 🔗Используемая документация:
> [Установка сервера FreeIPA](https://www.altlinux.org/FreeIPA/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0_FreeIPA)