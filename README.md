# Guideline и скрипты для подготовки сервера под Node.js + MongoDB + Redis

#### Перед началом работы, не зависимо от ОС нужно закрыть доступ на ваш сервер от *root*-а и создать нового пользователя под которым будет осуществляться вход/деплой и другие операции.
Но первым делом лучше обновиться:
```sh
sudo apt update && sudo apt upgrade
```
Теперь можно создавать пользователя.
```sh
# создаём пользователя и отвечаем на вопросы, которые будут задаваться
adduser имя_пользователя
```

В некоторых дистрибутивах Linux не доступна команда `adduser`.
Тогда используем более низкоуровневую команду:
```sh
# создаём пользователя, добавляем его в sudo и ставим bash оболочкой по-умолчанию
useradd --groups sudo --shell /bin/bash --create-home имя_пользователя
```
и задаём пароль для созданного пользователя:
```sh
passwd имя_пользователя
```

Тепереь нужно добавить созданного пользователя в *sudo*.
Можно об этом почитать [здесь](https://www.8host.com/blog/redaktirovanie-fajla-sudoers-v-ubuntu-i-centos/).
```sh
adduser имя_пользователя sudo
```
Если не доступна команда `adduser`, то можно сделать через **visudo**.
Для открытия файла */etc/sudoers* вводим:
```sh
sudo visudo
```
нужно добавить строку `имя_пользователя ALL=(ALL) ALL` и сохраниться.

На крайний случай можно сделать так:
```sh
echo 'имя_пользователя ALL=(ALL) ALL' >> /etc/sudoers
```

Теперь нужно выполнить базовые настройки безопасности, открываем *sshd_config*:
```sh
nano /etc/ssh/sshd_config
```
И меняем/добавляем строки:
- Отключить авторизацию от *root*-а заменив строку `PermitRootLogin yes` на `PermitRootLogin no`
- Разрешить подключение по *ssh* только созданному пользователю добавив/заменив строку `AllowUsers имя_пользователя`
- Запретить использование файлов *.rhosts* добавив/заменив строку `IgnoreRhosts yes`
- Запретить кросс-хостинговую аутентификацию (Host-Based Authentication) добавив/заменив строку `HostbasedAuthentication no`

Если требуется более тонкая настройка безопасности, то можно почитать [это](http://rus-linux.net/nlib.php?name=/MyLDP/sec/openssh.html).

Далее нужно рестартовать сервис *ssh*:
```sh
systemctl restart sshd
```
Так же рестарт сервиса *ssh* можно сделать командами `service sshd restart` или `/etc/init.d/sshd restart`
в зависимости от ОС и какие команды доступны.
Выходим из под *root*-а и заходим под созданным пользователем.

#### Если не хочется каждый раз при подключении к серверу вводить пароль, то можно добавить на сервер свой *ssh-ключ*.
Для копирования *ssh-ключ*-а лучше использовать `ssh-copy-id`([подробнее](http://xgu.ru/wiki/ssh-copy-id)).
На локальном компьютере переходим в папку с ключами:
```sh
cd ~/.ssh
```
и содаём ключ для подключения к нашему серверу:
```sh
ssh-copy-id -i id_rsa.pub имя_пользователя@хост
```
Так же для безопасности лучше сменить права у папки где хранятся *ssh-ключ*-и.
На сервере вводим:
```sh
sudo chmod -R 700 ~/.ssh
```
Если нужно полностью отключить вход по паролю (**не рокомендуется**) и оставить только вход по *ssh-ключу*,
то:
```sh
# открываем файл
nano /etc/ssh/sshd_config
```
меняем строку `PasswordAuthentication yes` на `PasswordAuthentication no` и рестартуем ssh-сервис:
```sh
sudo systemctl restart sshd
```
#### Настройка *firewall*-а
В данном примере показывается настройка *firewall*-а **ufw**.
Если он не установлен (для проверки введите например `sudo ufw status`), то устанавливаем:
```sh
sudo apt install ufw
```
Добавляем *OpenSSH* профиль в *firewall*:
```sh
sudo ufw allow 'OpenSSH'
```
Теперь нужно запустить *firewall*:
```sh
sudo ufw enable
```

#### Настройка времени на сервере используя протокол NTP
Открываем файл с настройками времени:
```sh
sudo nano /etc/systemd/timesyncd.conf
```
В этом файле нужно добавить/раскомментировать:
```
[Time]
NTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
FallbackNTP=0.pool.ntp.org 1.pool.ntp.org 0.fr.pool.ntp.org
```
После выполнить:
```sh
sudo timedatectl set-ntp true
```
И проверить статус:
```sh
timedatectl status
```
Настройка времени через **systemd-timesyncd** доступна в относительно свежих дистрибутивах Linux (в тех где присутствует *Systemd*).
В более старых придётся устанавливать **ntpdate** и [настраивать](https://help.ubuntu.ru/wiki/%D1%80%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE_%D0%BF%D0%BE_ubuntu_server/%D1%81%D0%B5%D1%82%D1%8C/ntp) через него.

#### Изменение имени компьютера
Меняем тут:
```sh
sudo nano /etc/hostname
```
и тут:
```sh
sudo nano /etc/hosts
```
и рестартуем сервер:
```sh
sudo reboot
```

#### Установка *NODE.js*
Для возможности использовать различные версии *NODE.js* нужно установить [NVM](https://github.com/nvm-sh/nvm).

Для установки потребуется [cURL](https://curl.haxx.se/):
```sh
sudo apt install curl
```
Устанавливаем *NVM*:
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```
После установки *NVM* нужно обновить сессию, достаточно закрыть и открыть заново терминал.

И устанавливаем последнюю LTS-версию *NODE.js*:
```sh
nvm install --lts
```

#### Установка и настройка *NGINX*
```sh
sudo apt install nginx
```
На всякий случай разблокируем сервис *nginx*
```sh
sudo systemctl unmask nginx.service
```
Чтобы посмотреть для каких приложений установлен профиль:
```sh
sudo ufw app list
```
Добавлем профили *nginx*:
```sh
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow 'Nginx HTTP'
```

#### Установка *Certbot* и создание сертификата для *NGINX*
[Официальная документация](https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx)
```sh
sudo apt install software-properties-common && sudo add-apt-repository universe && sudo add-apt-repository ppa:certbot/certbot
```
```sh
sudo apt install certbot python3-certbot-nginx
```

Перед добавлением сертификата для доменов должен быть правильно настроен **DNS**.
**A/AAAA** записи **DNS**-сервера должны содержать *IP-адрес* вашего сервера и только потом выполнить:
```sh
# можно сразу перечислить все домены
sudo certbot --nginx -d домен.com -d www.домен.com -d домен.ru -d www.домен.ru
```
после этого появится сгенерированный конфиг с именем `default` в папке `/etc/nginx/sites-available`,
но лучше его модифицировать под свои нужды.
Создать конфиг можно на сервисе [nginxconfig.io](https://nginxconfig.io).
Либо можно взять за основу конфиг в папке [nginx](nginx/).

Проверяем конфиг на наличие ошибок: `sudo nginx -t`

Перезагружаем *NGINX*: `sudo systemctl reload nginx`

Сертификаты Let’s Encrypt’s валидны всего [90 дней](https://letsencrypt.org/2015/11/09/why-90-days.html).
Для автообновления сертификатов нужно выполнить:
```sh
sudo systemctl enable certbot.timer
```
Теперь добавляем рестарт *NGINX*-а после каждого обновления сертификатов:
```sh
# создаём файл с хуками, которые будут выполняться после каждого автообновления
sudo touch /etc/letsencrypt/renewal-hooks/post/reload-services.sh
```
и прописываем там:
```sh
#!/bin/sh
systemctl reload nginx
```
так же нужно установить права для хука:
```sh
chmod 750 /etc/letsencrypt/renewal-hooks/post/reload-services.sh
```
и стартуем сервис автообновления:
```sh
sudo systemctl start certbot.timer
```
расписание автообновлений можно посмотреть набрав `systemctl list-timers certbot.timer`.

Если нужно вручную обновить сертификаты, то вводим:
```sh
sudo certbot renew --post-hook "systemctl reload nginx"
```

Настраиваем права для папки, где будут храниться сайты: `sudo chown -R $USER:$USER /var/www`
и создаём папку `/var/www/домен/public`, в которую будем складывать статику для *NGINX*.

#### Установка и настройка MongoDB
Установка MongoDB 4.2 Community Edition на Ubuntu 20 взята с [официальной документации](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).
Если нужно установить на другие ОС, то используем [официальный tutorial](https://docs.mongodb.com/manual/installation/#mongodb-community-edition-installation-tutorials).

> Конфиг *mongodb* по умолчанию лежит в `/etc/mongod.conf`, для MacOS в `/usr/local/etc/mongod.conf`
> Логи по умолчанию лежат в папке `/var/log/mongodb`
> БД по умолчанию лежит в папке `/var/lib/mongodb`

##### Создание и настройка репликации
```sh
# создаём папки для БД и арбитра
sudo mkdir /data && sudo mkdir /data/rs0 && sudo mkdir /data/rs1 && sudo mkdir /data/arbiter0
```

Меняем владельца на *mongodb:mongodb* от которой будет запускаться *БД*:
```sh
sudo chown -R mongodb:mongodb /data/
```

Генерируем keyFile для реплик (если Permission Denied, то из под root-а):
```sh
openssl rand -base64 756 > /etc/mongod.keyfile
chown mongodb:mongodb /etc/mongod.keyfile
chmod 400 /etc/mongod.keyfile
```

Заменяем конфиг `/etc/mongod.conf` на [mongod.conf](mongodb/mongod.conf) и запускаем все серверы реплик:
<!-- sudo mongod --dbpath /data/rs0 --config /etc/mongod.conf --fork -->
```sh
sudo mongod --dbpath /data/rs0 --port 27017 --config /etc/mongod.conf --fork
```
```sh
sudo mongod --dbpath /data/rs1 --port 27018 --config /etc/mongod.conf --fork
```
```sh
sudo mongod --dbpath /data/arbiter0 --port 27019 --config /etc/mongod.conf --fork
```
Подключаемся к одному из серверов (только не к арбитру):
```sh
mongo --port 27017
```
и выполняем:
```sh
rs.initiate()
```
После добавляем остальные реплики:
```sh
rs.add('localhost:27018')
rs.addArb('localhost:27019')
```

Теперь нужно проверить, что репликация работает:
```sh
use test
db.users.insertOne({ first_name: 'John', last_name: 'Smith' })
```

подключаемся к реплике (можно в соседней вкладке):
```sh
mongo --port 27018
```
и выполняем:
```sh
rs.slaveOk()
use test
db.users.find()
```

Создание пользователей (**нужно выполнять на master**-е):
```sh
use admin
```
```sh
# создание основных пользователей, роль 'userAdminAnyDatabase' позволяет только создавать пользователей и назначать им роли
db.createUser({user: "admin", pwd: "пароль", roles: [{ role: "userAdminAnyDatabase", db: "admin" }]})
```
```sh
db.createUser({user: "root", pwd: "пароль", roles: [{ role: "root", db: "admin" }]})
```

```sh
# создание пользователя для вашей БД
db.createUser({user: "пользователь", pwd: "пароль", roles: [{ role: "readWrite", db: "имя_БД" }]})
```

Закрываем все серверы БД, но **сначала нужно выключить арбитра!**:
```sh
# подключаемся к арбитру
mongo --port 27019
use admin
db.shutdownServer()
```
```sh
# подключаемся к реплике №1
mongo --port 27017
use admin
db.shutdownServer()
```
```sh
# подключаемся к реплике №2
mongo --port 27018
use admin
db.shutdownServer()
```

В файле `/etc/mongod.conf` нужно раскомментировать весь блок `security`.

Настраиваем *systemd* для реплик:
Создаём/заменяем файл `/lib/systemd/system/mongod.service` на [mongod.service](mongodb/mongod.service)
```sh
sudo nano /lib/systemd/system/mongod.service
```
а так же копируем файлы [mongod-rs0.service](mongodb/mongod-rs0.service), [mongod-rs1.service](mongodb/mongod-rs1.service)
и [mongod-arbiter0.service](mongodb/mongod-arbiter0.service) в каталог `/lib/systemd/system/`.

Перезапускаем демона *systemd*:
```sh
sudo systemctl daemon-reload
```

```sh
# запуск сервиса
sudo systemctl start mongod
```

```sh
# добавляем запуск сервера после перезагрузки системы
sudo systemctl enable mongod mongod-rs0 mongod-rs1 mongod-arbiter0
```