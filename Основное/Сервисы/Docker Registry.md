> [!INFO] Что такое **Docker Registry**?
> Централизованный сервис или серверное приложение для хранения, управления и распространения образов Docker, работающее как "склад" контейнеров, куда их загружают и откуда скачивают

Мы будем работать с ним на `ALT Linux p11`,  основные используемые дистрибутивы:
- `docker-engine`
- `docker-compose-v2`
- `apache2`

Установка дистрибутивов:
```bash
apt-get update && apt-get install docker-engine docker-compose-v2 apache2 -y
```
Создание директории для работы:
```bash
mkdir -p docker-registry/{auth, data}
cd docker-registry
```
Изменяем файлик по пути `/etc/docker/daemon.json`:
```bash
{
"storage-driver": "overlay2",
"insecure-registries": ["localhost:5800"]
}
```
Перезапускаем docker:
```bash
systemctl restart docker
```
Создаем файл в ранее созданной папке, `docker-registry/docker-compose.yml`:
```yml
version: '3.8'

services:
	registry:
		image: registry:2
		container_name: secured-registry
		restart:always
		ports:
		  - "5800:5000"
		volumes:
		  - ./data:/var/lib/registry
		  - ./auth:/auth
		  - ./config.yml:/etc/docker/registry/config.yml 
	    
	registry-ui:
		image: joxit/docker-registry-ui:1.5-static
		container_name: registry-ui
		restart: always
		ports:
		  - "8080:80"
		environment:
			REGISTRY_URL: http://localhost:5800
			SINGLE_REGISTRY_MODE: "true"
			SHOW_CONTENT_DIGEST: "true"
		 depends_on:
		   - registry
volumes:
	registry:
```
Создаем конфигурационный файл, `docker-registry/config.yml`:
```yml
version: 0.1
log:
	level: info

storage:
	filesystem:
		rootdirectory: /var/lib/registry
	delete:
		enabled: true

auth:
	htpasswd:
		realm: Registry Realm
		path: /auth/htpasswd

http:
	addr: :5000
	headers:
		X-Content-Type-Options: [nosniff]

health:
	storagedriver:
		enabled: true
		interval: 10s
		threshold: 3
```
Создаем пользователей:
```bash
htpasswd -Bc auth/htpasswd alexey # администратор с именем alexey
htpasswd -B auth/htpasswd user # обычный пользователь с именем user
```
Работа с контейнером:
```bash
docker compose up -d # запуск контейнера
docker compose down # остановка контейнера
docker compose logs -f # логи
```
Проверка работоспособности:
```bash
curl http://localhost:5800/v2
$ UNAUTHORIZED
curl -u user http://localhost:5800/v2/_catalog
$ Enter host password for user 'user':
docker login localhost:5800
$ Username (admin): alexey
$ Password: P@ssw0rd

$ Login Succeeded
```

На клиентах устанавливается следующие дистрибутивы:
```bash
apt-get update && apt-get install docker-cli docker-compose-v2 -y
```
Основные команды для клиентов:
```bash
docker ps
docker pull
docker push
docker compose up
```
