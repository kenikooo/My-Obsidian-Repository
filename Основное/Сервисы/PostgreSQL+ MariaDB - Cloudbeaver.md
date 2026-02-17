— Перейти в [[HUB]] —
> [!INFO] Что это за **смесь**?
Базы данных PostgreSQL и MariaDB + веб-версия инструмента управления базами данных DBeaver. 

> [!WARNING] Предупреждение
> Вся работа выполнялась на ALT Linux Server 10.4, в качестве подключения использовался сетевой мост

> [!INFO] Используемые дистрибутивы
```bash
apt-get install [название пакета] -y
> docker-engine
> postgresql17
> postgresql17-server
> mariadb
```
### Базы данных PostgreSQL и MariaDB
### > PostgreSQL
> [!BUG] Перед запуском службы обязательно
```bash
> /etc/init.d/postgresql initdb
> systemctl enable --now postgresql
```
Вход в PostgreSQL
```bash
> psql -U postgres
```
Создание БД и пользователя
```bash
> CREATE DATABASE НАЗВАНИЕ_БД;
> CREATE USER ИМЯ_ПОЛЬЗОВАТЕЛЯ WITH PASSWORD 'ПАРОЛЬ';
> GRANT ALL PRIVILEGES ON DATABASE НАЗВАНИЕ_БД TO ИМЯ_ПОЛЬЗОВАТЕЛЯ;
> exit
```
Инициализация без пароля
```bash
su -l postgres -s /bin/sh -c "/usr/bin/initdb --pgdata=/var/lib/pgsql/data"
```
- `-l` - выполнение команды под другим пользователем с сохранением текущего сеанса;
- `-s` - указываем командную облочку;
### > MariaDB
>[!BUG] Запуск службы
```
systemctl enable --now mariadb
> После этого производим первоначальную настройку:
mysql_secure_installation
```

Вход в MariaDB
```
mysql -u root -p
```
Создаем БД и пользователя
```bash
> CREATE DATABASE НАЗВАНИЕ_БД;
> CREATE USER 'ИМЯ_ПОЛЬЗОВАТЕЛЯ'@'%' IDENTIFIED BY 'ВАШ ПАРОЛЬ';
> GRANT ALL PRIVILEGES ON НАЗВАНИЕ_БД.* TO 'ИМЯ_ПОЛЬЗОВАТЕЛЯ'@'%';
> FLUSH PRIVILEGES;
```
Изменим файлик /et
c/my.cnf, добавив в конец следующие строки:
```bash
[mysqld]
skip-networking=0
skip-bind-address
```
### Cloudbeaver
> [!VERIFY] Запуск службы для работы с **Docker**
```bash
> systemctl enable --now docker
```

> [!WARNING] Запуск CloudBeaver
```
Для начала обязательно: 
> touch docker-compose.yml
```
Содержимое файла:
```bash
version: '3.8'

services:
  cloudbeaver:
    image: dbeaver/cloudbeaver:latest
    container_name: cloudbeaver
    ports:
      - "8978:8978"
    volumes:
      - cloudbeaver-data:/opt/cloudbeaver/workspace
    restart: unless-stopped

volumes:
  cloudbeaver-data:
```

>[!BUG] Запуск Cloudbeaver через **docker run**
```bash
> docker run -d \
> --name cloudbeaver \
> -p 8978:8978 \
> -v /opt/cloudbeaver-data:/opt/cloudbeaver/workspace \
> dbeaver/cloudbeaver:latest
```
После этого можно перейти в браузер: 
```
> http://<IP-Сервера>:8978
```
