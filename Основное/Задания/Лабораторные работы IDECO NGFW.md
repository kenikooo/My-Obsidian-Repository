# Лабораторная работа 1. Установка и первоначальная настройка Ideco NGFW Novum

Соберите виртуальный стенд в VirtualBox в соответствие с предложенной топологией:

- **idc-fw**: [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum") 21
- **idc-srv**: Альт Сервер 11 (в минимальной установке)
- **idc-cli**: Альт Рабочая станция 11 или Simply Linux 11 (на выбор)
- **Host-PC**: ПК на котором запущен VirtualBox
![[Pasted image 20260317092607.png]]
Выполните первоначальную настройку по следующему плану:

1. Зарегистрируйтесь в [MY.IDECO](https://my.ideco.ru/). Подробнее в статье [О личном кабинете MY.IDECO](https://docs.ideco.ru/my-ideco/ru/my/index.html).
2. ~~Скачайте загрузочный образ продукта. Дистрибутивы также доступны в личном кабинете MY.IDECO.~~
3. Определитесь с устройством, на которое собираетесь устанавливать [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum"), и выполните предварительные действия. [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum") можно устанавливать как на [виртуальную машину](https://docs.ideco.ru/v21/ru/ngfw/installation/specifics-of-hypervisor-settings), так и на [физический сервер](https://docs.ideco.ru/v21/ru/ngfw/installation/usb).
4. Установите [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum") на устройство, создайте учетную запись администратора. Для этого воспользуйтесь статьей [Установка](https://docs.ideco.ru/v21/ru/ngfw/installation/installation-process).
5. Выполните первоначальную настройку [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum"). Воспользуйтесь статьей [Первоначальная настройка](https://docs.ideco.ru/v21/ru/ngfw/installation/initial-setup).
6. Зарегистрируйте сервер в [MY.IDECO](https://my.ideco.ru/) и получите лицензию.
7. Создайте учетные записи пользователей и настройте авторизацию, чтобы получить доступ в интернет через [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum"). Более подробная информация представлена в статье [Получение доступа в интернет](https://docs.ideco.ru/v21/ru/ngfw/installation/get-internet).

- для доступа в сеть Интернет **idc-srv** должен быть авторизован на **idc-fw** по IP
- для доступа в сеть Интернет **idc-cli** должен быть авторизован на **idc-fw** по подсети

  
# Лабораторная работа 2. Интеграция Ideco NGFW Novum с Active Directory/Samba DC

На виртуальной машине **idc-srv** разверните контроллер домена Samba/DC или FreeIPA на выбор
Введите в развёрнутый домен виртуальные машины **idc-fw** и **idc-cli**
Создайте в домене минимум **две** группы безопасности, с минимум **тремя** пользователями в каждой
Выполните импорт пользователей из домена в [Ideco NGFW Novum](https://moodle.space-bpk.ru/mod/resource/view.php?id=109 "Ideco NGFW Novum")

# Лабораторная работа 3. Установка, настройка и использование Ideco Client

На виртуальную машину **idc-cli** выполните установку Ideco Client.
На виртуальной машине **idc-cli** выполните настройку Ideco Client, а именно создайте профиль для авторизации из-под любого доменного пользователя, проверьте работоспособность авторизации.

**Дополнительно**:
Выполните [кастомную настройку Ideco Client](https://docs.ideco.ru/v21/ru/ngfw/settings/users/ideco-client/custom-settings) для SSO, запретив пользователю создание новых профилей.  
SSO - пользователь для подключения использует данные системы.