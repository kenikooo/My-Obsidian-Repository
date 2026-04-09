— Перейти в [[HUB]] —
> [!INFO] Что такое **Ansible**?
Инструмент с открытым исходным кодом для автоматизации управления конфигурациями, развёртывания приложений и оркестрации инфраструктуры.

---
### Используемые дистрибутивы

> [!tldr] **Необходимые пакеты**
> 
> - `ansible` — основной пакет
> - `sshpass` — для авторизации по паролю в SSH (опционально)

### Основные каталоги и файлы

> [!info] **Структура файлов**
> 
> - `/etc/ansible/` — главный каталог
> - `ansible.cfg` — конфигурационный файл
> - `hosts` — файл с группами/объектами по умолчанию
> - `group_vars/` — переменные для групп хостов
> - `host_vars/` — переменные для конкретных хостов
> - `group_vars/all.yml` — переменные для всех хостов

Ansible использует SSH для выполнения команд на удалённых машинах. Важно настроить пользователя и способ аутентификации (пароль или ключ SSH).

---

### Основные команды

```bash
# Выполнение команд
ansible <group | object> -m <module> -a <arguments>

# Проверка доступности хостов
ansible all -m ping

# Выполнение команд в удалённой оболочке
ansible all -m shell -a 'uptime'

# Копирование файлов
ansible all -m copy -a 'src=/local/path dest=/remote/path'

# Получение информации о системе
ansible all -m setup -a 'filter=ansible_distribution'
```
Запуск Playbook:
```bash
ansible-playbook -i [Инвентарь] имя_плейбука.yml

# Пример
ansible-playbook -i inventory.ini monitoring.yml
```

---

### Простой Ansible.cfg
```bash
[defaults]
inventory = ./inventory.ini
host_key_checking = False
```
**Пояснение**: `[defaults]` - это заголовок секции, все параметры, идущие ниже, являются настройками "по умолчанию" для работы Ansible в текущей директории;
`inventory = ./inventory.ini` - строка указывает Ansible, где именно искать список серверов;
`host_key_checking = False` - это настройка безопасности SSH, которая сильно упрощает жизнь в локальных сетях. Как это работает? Если 
`True`: при первом подключении по SSH система спрашиваешь "Вы уверены что хотите доверять этому узлу?" и просит подтверждения. Если сервер переустановили и ключ изменился, SSH заблокирует соединение;
`False`:  Ansible игнорирует проверку ключей хоста. Он не будет спрашивать подтверждения при первом подключении и не упадет с ошибкой, если ключ сервера изменился.
Зачем это нужно: полезно для автоматизации, чтобы скрипт не "подвис" в ожидании ответа `yes` в консоли
### Инвентарь (inventory)

> [!tip] **Файл hosts** По умолчанию Ansible использует `/etc/ansible/hosts`, но можно задать другой, указав `-i` в команде.

#### **Пример inventory.yml**

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.33.11
          ansible_user: root
        web2:
          ansible_host: 192.168.33.12
          ansible_user: root
```

#### **Пример inventory.ini**
```ini
[haproxy_nodes]
Cloud-HA01 ansible_host=192.168.10.65
Cloud-HA02 ansible_host=192.168.10.66

[web_nodes]
Cloud-WEB01 ansible_host=192.168.10.67
Cloud-WEB02 ansible_host=192.168.10.68

[monitoring]
cloud-mon

[target_nodes]
cloud-adm

[all:vars]
ansible_user=altlinux
```
**Пояснение**: `[haproxy_nodes]`, `[web_nodes]` и тд - группы. Деление на группы происходит, чтобы в плейбуке сказать "Установи Nginx только на `[web_nodes]`";
`Cloud-HA01`, `Cloud-WEB01` - это Inventory Name. Имя сервера внутри Ansible;
`ansible_host=192.168.10.65` - переменная, указывающая реальный IP или DNS сервер для подключения;
`cloud-mon` - ansible_host не указан, поэтому ansible будет пробоваться подключиться напрямую по имени (оно должно резолвиться через /etc/hosts или DNS)
`[all:vars]` - блок общих переменных для всех групп в этом файле:
- `ansible_user=altlinux` - говорит Ansible заходить на все серверы под пользователем `altlinux`

---

### Переменные групп и хостов

> [!example] **Пример `group_vars/all.yml`**

```yaml
ansible_user: admin
ansible_python_interpreter: /usr/bin/python3

# Авторизация по паролю
ansible_ssh_user: user
ansible_ssh_pass: password

# Авторизация по ключу
ansible_ssh_private_key_file: /home/user/.ssh/id_rsa
```

---

### Playbook (сценарии автоматизации)

> [!quote] **Playbook** — это последовательность шагов, выполняемых на удалённых серверах.

#### **Простейший playbook**

```yaml
---
- name: Установка пакетов
  hosts: webservers
  become: yes  # Запуск от имени root
  tasks:
    - name: Обновить кеш пакетов
      apt:
        update_cache: yes

    - name: Установить Nginx
      apt:
        name: nginx
        state: present
```

#### **Запуск**

```bash
ansible-playbook install_nginx.yml
```

---

### Jinja2 (Шаблоны конфигураций)

> [!note] **Jinja2** — шаблонный движок, позволяющий использовать переменные и логику в файлах конфигурации.

#### **Пример шаблона** `nginx.conf.j2`

```jinja2
server {
    listen 80;
    server_name {{ ansible_fqdn }};
    location / {
        proxy_pass http://localhost:8080;
    }
}
```

#### **Пример использования в playbook**

```yaml
- name: Создание конфигурации Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
```

---

### 🔗 Полезные ссылки

- 📖 [Документация Ansible](https://docs.ansible.com/)
- 📰 [Обучение на Habr](https://habr.com/ru/articles/305400/)

