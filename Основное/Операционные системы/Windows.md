— Перейти в [[HUB]] —
Содержание:
1. [[#Настройка NAT через PowerShell]]
2. [[#Получение списка дисков]]
3. [[#Просмотр разделов на дисках]]
4. [[#Удаление всех разделов на диске 0]]
5. [[#Обход ввода MS Account]]
### Настройка NAT через PowerShell
**Проверка состояния NAT:**  
```bash
Get-NetNat
```
*Если ничего не выводит, значит NAT отключен.*  

**Создание правила NAT:**  
```bash
> New-NetNat -Name "InternalNAT" -InternalIPInterfaceAddressPrefix 192.168.100.0/24 
> Remove-NetNat -Name "InetnalNAT"
```

Включить ответ на ping:
```bash
netsh advfirewall firewall add rule name="Allow ICMPv4" dir=in action=allow protocol=icmpv4
```
Проверить, что правило применилось:
```bash
netsh advfirewall firewall show rule name="Allow ICMPv4"
```

---
### Получение списка дисков  
```bash
Get-PhysicalDisk | Select FriendlyName, DeviceId
```
*Выводит список дисков с их именами и ID.*  

### Просмотр разделов на дисках  
```bash
Get-Partition | Select DiskNumber, PartitionNumber, DriveLetter, Size
```
*Показывает номера дисков, разделов, буквы и размеры.*  

### Удаление всех разделов на диске 0  
```bash
Get-Partition -DiskNumber 0 | Remove-Partition -Confirm:$false
```
*Без подтверждения удаляет все разделы с диска 0.*  

**После выполнения команд объедините дисковое пространство через**  управление дисками (Disk Management).
Запуск:
```bash
Win + X → Управление дисками или diskmgmt.msc 
```
---
### Обход ввода MS Account
Используем в ВМ сочетание клавиш Shift + F10 (на ноутбуках добавится FN)
После чего, откроется консоль под администратором, в ней мы прописываем следующее:
```bash
start ms-cxh:localonly
```
И создаем локальную учетную запись.