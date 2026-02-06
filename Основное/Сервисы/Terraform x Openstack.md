Содержимое ~/.terraformrc
```bash
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```
Далее, создаем и заполняем файл: /home/altlinux/project02/provider.tf
```bash
terraform {
	required_providers {
		openstack = {
			source = "terraform-provider-openstack/openstack"
			version = "2.1.0"
		}
	}
}

provider "openstack" {
	auth_url = "https://cyberinfra1.ssa2026.region:5000/v3"
	tenant_name = "Porject02"
	user_name = "Competitor02"
	password = "ПАРОЛЬ ДЛЯ ВХОДА"
	insecure = true
}
```
В той же директории создаем cloudinit.conf
```bash
export OS_AUTH_URL=https://cyberinfra1.ssa2026.region:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_TYPE=password
export OS_PROJECT_DOMAIN_NAME=Competence_SiSA
export OS_USER_DOMAIN_NAME=Competence_SiSA
export OS_USERNAME=Competitor02
export OS_PASSWORD=ВАШ ПАРОЛЬ
export OS_PROJECT_NAME=Project02
```
Обязательные команды после написания
```bash
source cloudinit.conf
# Проверка
openstack --insecure server list # если выводит ошибку, попробуйте пинг до сайта, если не идет, дополняем /etc/hosts
```
После этого, выполняем инициализацию terraform с openstack
```bash
terraform init
```
Запуск из любой директории, скрипт deploy-cloudinfra.sh
```bash
cd /home/altlinux/bin/terraform-project
terraform apply -auto-approve

```
Снос после terraform apply:
```bash
terraform destroy
```
### Первоначальная настройка при развертке cloud-init.yml
```bash
#cloud-config
chpasswd:
  expire: false
  users:
    - {name: altlinux, password: P@ssw0rd, type: text}
    - {name: root, password: toor, type: text}
  ssh_pwauth: false

users:
  - name: altlinux
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: wheel
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAB3NzaC1yc2EAAAADAQABAAABgQDcMzrUUezuP57DpmOs5cirucIhyn/IbxWJvfxTUk3jwJruWgsK/B+otQjUbE2uC73VzPMxrUvsJE7qFHEyHfl4zZOs4tlC102oxUpoelUBpqRmIPgQagd8gphvoZ9LolSwULiRBpzoG8/2BNBlxePIJCQrhB5LK5LwGJ17yhzIYZiS7M1j1DwmcOnE9jNIBDsmJEgUOBKRi/O5E3S8hvQUGr8f6LPq7aYGaXFMihdYO+dbPIT9J5m9R7hmsald/lS/GR3kEsP+ghaWHDYW15EtxWKTow/n8nRUadbPm/HcWm+TknMrPakrelRo2xZ2zHx84gQ47H8wQwV10v5YcMQ19B7KSAnbMeBDheT2MhbT+elhA1RYVtOUoUnDP+izNvavgfQurPx6bVEZ7JCTbjjC2ljGSeQOnUddYufELWIwNJ/dMdQNNYzmuKk6ZfItbNM6FHJbNamMiWhcFYElIZfRPyJdLc0Xqv0sndD7cbtxvQfssvmgqq8rXEhmMkcVO9U= altlinux@cloud-adm
```
### Написание variable.tf
```bash
variable "image_id" {
	type    = string
	default = "ca7c0b8b-3d17-47c6-9c4b-2d3871560a20"
}
variable "external_net" {
  type    = string
  default = "6c32f736-636d-4023-ae43-e41180886711"
}
variable "external_sub" {
  type    = string
  default = "14bb9e6a-b88c-4361-8009-ce405df969be"
}
```
### Создание сети и подсети
```bash
# Internal Network
resource "openstack_networking_network_v2" "network_internal" {
  name            = "Internal-Net"
}
resource "openstack_networking_subnet_v2" "subnet_internal" {
  name        = "Internal-Subnet"
  network_id  = openstack_networking_network_v2.network_internal.id
  cidr        = "192.168.10.0/26"
  ip_version  = 4
  dns_nameservers = ["8.8.8.8"]
}

# Management Network
resource "openstack_networking_network_v2" "network_management" {
  name            = "Management-Net"
}
resource "openstack_networking_subnet_v2" "subnet_management" {
  name          = "Mgmt-Subnet"
  network_id    = openstack_networking_network_v2.network_management.id
  cidr          = "192.168.10.64/26"
  ip_version    = 4
  dns_nameservers = ["8.8.8.8"]
}

# Backup Network
resource "openstack_networking_network_v2" "network_backup" {
  name            = "Backup-Net"
}
resource "openstack_networking_subnet_v2" "subnet_backup" {
  name        = "Backup-Subnet"
  network_id  = openstack_networking_network_v2.network_backup.id
  cidr        = "192.168.10.128/27"
  ip_version  = 4
  dns_nameservers = ["8.8.8.8"]
}
```
### Создание портов
```bash
resource "openstack_networking_port_v2" "port_management_01" {
	name	= "port_mgmt-ha01"
	network_id = openstack_networking_network_v2.network_management.id
	admin_state_up  = "true"
	fixed_ip {
		subnet_id = openstack_networking_subnet_v2.subnet_management.id
		ip_address = "192.168.10.65"
	}
}

resource "openstack_networking_port_v2" "port_internal_01" {
	name	= "port_int-ha01"
	network_id = openstack_networking_network_v2.network_internal.id
	admin_state_up  = "true"
	fixed_ip {
		subnet_id = openstack_networking_subnet_v2.subnet_internal.id
		ip_address = "192.168.10.1"
	}
}

resource "openstack_networking_port_v2" "port_external_01" {
	name	= "port_ext-ha01"
	network_id = var.external_net
	admin_state_up  = "true"
	fixed_ip {
                subnet_id = var.external_sub 
		ip_address = "172.16.1.4"
	}
}
```
### Создание групп безопасности ICMP и SSH
```bash
resource "openstack_networking_secgroup_v2" "icmp" {
	name = "icmp"
}
resource "openstack_networking_secgroup_rule_v2" "icmp" {
	direction = "ingress"
	ethertype = "IPv4"
	protocol = "icmp"
	remote_ip_prefix = "0.0.0.0/0"
	security_group_id = openstack_networking_secgroup_v2.icmp.id
}
resource "openstack_networking_secgroup_v2" "ssh" {
	name = "ssh"
}
resource "openstack_networking_secgroup_rule_v2" "ssh" {
	direction = "ingress"
	ethertype = "IPv4"
	protocol = "tcp"
	port_range_min = 22
	port_range_max = 22
	remote_ip_prefix = "172.16.1.14/32" # разрешаем подключение только для Cloud-ADM
	security_group_id = openstack_networking_secgroup_v2.ssh.id
}
```
### Создание инстанса
 ```bash
resource "openstack_compute_instance_v2" "cloud_ha_01" {
	name		= "Cloud-HA01"
	flavor_id	= "100"
	config_drive = true # Если машина не может запуститься и плюет ошибками (см. ниже), указывается данная настройка
	user_data = file("cloud-init.yml") 
	network {
		port = openstack_networking_port_v2.port_external_01.id
	}
	network {
		port = openstack_networking_port_v2.port_management_01.id
	}
	network {
		port = openstack_networking_port_v2.port_internal_01.id
	}
	security_groups = [
		openstack_networking_secgroup_v2.icmp.name
		openstack_networking_secgroup_v2.ssh.name
	]
	block_device {
		uuid			= var.image_id
		source_type		        = "image"
		volume_size		        = 10
		boot_index		        = 0
   	        destination_type      = "volume"
		delete_on_termination	= true
	}
}
 ```
 >[!ERROR] Вывод лога
 >`[ 20.215005] cloud-init[604]: 2026-02-06 05:16:29,915 - url_helper.py[WARNING]: Calling 'http://[fd00:ec2::254]/2009-04-04/meta-data/instance-id' failed [39/240s]: request error [HTTPConnectionPool(host='fd00:ec2::254', port=80): Max retries exceeded with url: /2009-04-04/meta-data/instance-id (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7fdb49a915e0>: Failed to establish a new connection: [Errno 101] Network is unreachable'))]`

### Настройка безопасности
```bash
resource "openstack_networking_secgroup_v2" "ha_sg" {
  name        = "HA-Security-Group"
  description = "Security group for HA nodes"
}

resource "openstack_networking_secgroup_rule_v2" "allow_icmp" {
  direction         = "ingress"
  ethertype         = "IPv4"
  protocol          = "icmp"
  security_group_id = openstack_networking_secgroup_v2.ha_sg.id
}

# Разрешаем SSH только из сети, где находится Cloud-ADM
resource "openstack_networking_secgroup_rule_v2" "allow_ssh_from_adm" {
  direction         = "ingress"
  ethertype         = "IPv4"
  protocol          = "tcp"
  port_range_min    = 22
  port_range_max    = 22
  remote_ip_prefix  = "192.168.10.0/24"
  security_group_id = openstack_networking_secgroup_v2.ha_sg.id
}
```

```bash
# 1. Создание Load Balancer
resource "openstack_lb_loadbalancer_v2" "lb_ha" {
  name          = "Load Balancer"
  vip_address   = "172.16.1.1"
  vip_subnet_id = openstack_networking_subnet_v2.subnet_external.id
}

# 2. Listener для HTTP (порт 80)
resource "openstack_lb_listener_v2" "listener_http" {
  name            = "Listener-HTTP"
  protocol        = "HTTP"
  protocol_port   = 80
  loadbalancer_id = openstack_lb_loadbalancer_v2.lb_ha.id
}

# 3. Listener для HTTPS (порт 443)
resource "openstack_lb_listener_v2" "listener_https" {
  name            = "Listener-HTTPS"
  protocol        = "HTTPS"
  protocol_port   = 443
  loadbalancer_id = openstack_lb_loadbalancer_v2.lb_ha.id
}

# 4. Пул балансировки
resource "openstack_lb_pool_v2" "pool_ha" {
  protocol    = "HTTP"
  lb_method   = "ROUND_ROBIN"
  listener_id = openstack_lb_listener_v2.listener_http.id
}

# 5. Добавление членов (Cloud-HA01 и Cloud-HA02)
resource "openstack_lb_member_v2" "cloud-ha01" {
  pool_id       = openstack_lb_pool_v2.pool_ha.id
  address       = "172.16.1.2"
  protocol_port = 80
  subnet_id     = openstack_networking_subnet_v2.subnet_external.id
}
resource "openstack_lb_member_v2" "member2" {
  pool_id       = openstack_lb_pool_v2.pool_ha.id
  address       = "172.16.1.3"
  protocol_port = 80
  subnet_id     = openstack_networking_subnet_v2.subnet_external.id
}
```

### 🔗 Используемая документация
> [Настройка .terraformrc](https://yandex.cloud/ru/docs/ydb/terraform/install?utm_referrer=https%3A%2F%2Fwww.google.com%2F](https://yandex.cloud/ru/docs/ydb/terraform/install?utm_referrer=https%3A%2F%2Fwww.google.com%2F)
> [Зеркало terraform](https://yandex.cloud/ru/docs/ydb/terraform/install?utm_referrer=https%3A%2F%2Fwww.google.com%2F#:~:text=%D0%BD%D0%B0%D1%88%D0%B8%D0%BC%20%D1%81%D0%BF%D0%B5%D1%86%D0%B8%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%20%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC-,%D0%B7%D0%B5%D1%80%D0%BA%D0%B0%D0%BB%D0%BE%D0%BC,-.%20%D0%A1%D0%BA%D0%B0%D1%87%D0%B0%D0%B9%D1%82%D0%B5%20%D0%B4%D0%B8%D1%81%D1%82%D1%80%D0%B8%D0%B1%D1%83%D1%82%D0%B8%D0%B2%20Terraform)
> [Terraform x Obsidian | Selectel Docs](https://docs.selectel.ru/en/terraform/quickstart/)
> [Environment Variables | cloudinit.yml](https://docs.openstack.org/python-openstackclient/latest/cli/authentication.html#:~:text=%24%20export%20OS_IDENTITY_API_VERSION%3D3%0A%24%20export%20OS_AUTH_URL%3Dhttp%3A//localhost%3A5000/v3%0A%24%20export%20OS_DEFAULT_DOMAIN%3Ddefault%0A%24%20export%20OS_USERNAME%3Dadmin%0A%24%20export%20OS_PASSWORD%3Dsecret%0A%24%20export%20OS_PROJECT_NAME%3Dadmin)
> [Ответ Google AI при поиске](https://www.google.com/search?q=provider+tf+terraform-openstack&oq=provider+tf+terraform-openstack&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIHCAEQIRigATIHCAIQIRigATIHCAMQIRifBTIHCAQQIRifBTIHCAUQIRifBTIHCAYQIRifBTIHCAcQIRifBTIHCAgQIRifBTIHCAkQIRifBdIBCDk2MTBqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8#:~:text=terraform%20%7B%0A%20%20required_providers%20%7B%0A%20%20%20%20openstack,OS_AUTH_URL%22%0A%20%20region%20%20%20%20%20%3D%20%22OS_REGION%22%0A%7D)
