— Перейти в [[HUB]] —
>[!WARNING] Установка на **ALT Linux**
>`apt-get update && apt-get install asterisk`

Настройка звонков по внутренней сети.
`/etc/asterisk/pjsip.conf` содержимое:
```bash
[transport-udp]
type = transport
protocol = udp
bind = 0.0.0.0:5060

[1001]
type = endpoint
context = internal
disallow = all
allow = alaw
allow = ulaw
auth = 1001
aors = 1001

[1001]
tpye = auth
auth_type = userpass
username = 1001
password = 1001P@ssw0rd

[1001]
type = aor
max_contacts = 1

[1002]
type = endpoint
context = internal
disallow = all
allow = alaw
allow = ulaw
auth = 1002
aors = 1002

[1002]
tpye = auth
auth_type = userpass
username = 1002
password = 1002P@ssw0rd

[1002]
type = aor
max_contacts = 1
```