> [!INFO] Что такое **Gitea**?
> Легковесный, бесплатный и открытый веб сервис для хостинга Git-репозиторием, который позволяет командам и отдельным разработчикам управлять кодом, сотрудничать, отслеживать задачи и выполнять код-ревью, выступая как само-хостируемая альтернатива GitHub или GitLab, с акцентом на простоту

Устанавливаем необходимые пакеты:
```bash
apt-get update && apt-get install git wget postgresql17-server postgresql17
```
Сразу же развернем базу данных, чтобы в будущем с ней не возникло проблем:
```bash
/etc/init.d/postgresql initdb # пароль запомните, он пригодится в будущем
```
И запустим службу:
```bash
systemctl enable --now postgresql
```
Для безопасности, создадим пользователя для работы с GIT:
```bash
useradd --system --shell /bin/bash --create-home --home-dir /home/git --comment 'Git Version Control' git
```
После чего, в целях безопасности для сервисов обычно используется заблокированный пароль, что можно сделать так:
```bash
passwd -l git
```
Создаем структуру каталогов, требуется создать директории для данных, логов и конфигурации и задать им правильные права:
```bash
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea
```
Далее, загружаем последнюю стабильную версию бинарного файла и сразу же переместим его в системную директорию:
```bash
wget -O /tmp/gitea https://dl.gitea.com/gitea/1.25.3/gitea-1.25.3-linux-amd64
mv /tmp/gitea /usr/local/bin/gitea
chmod +x /usr/local/bin/gitea
```
Создадим файл службы для автозапуска GitEA:
```bash
vim /etc/systemd/system/gitea.service
```
>[!BUG] Содержимое gitea.service
```bash
[Unit]
Description=Gitea (Git with a cup of tea)
After=network.target

[Service]
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```
Следующим шагом нам нужно активировать и запустить службу:
```bash
systemctl daemon-reload
systemctl enable --now gitea
systemctl status gitea # должен вывести Active и никаких ошибок
```
Затем, переходим в браузер:
```bash
http://IP-SERVER:3000
В качестве юзера и базы данных:
> postgres
пароль: тот, что Вы задали при установке postgres
```