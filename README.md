# Guideline и скрипты для подготовки сервера под Node.js + MongoDB + Redis

#### Перед началом работы, не зависимо от ОС нужно закрыть доступ на ваш сервер от *root*-а и создать нового пользователя под которым будет осуществляться вход/деплой и другие операции.
Подробнее про опции можно прочитать [тут](https://linux.die.net/man/8/useradd)
```sh
# создаём пользователя, добавляем его в sudo и ставим bash оболочкой по-умолчанию
useradd --groups sudo --shell /bin/bash --create-home имя_пользователя
```

В некоторых дистрибутивах Linux доступна более высокоуровневая
команда `adduser`. Если она присутствует, то лучше воспользоваться ей:
```sh
# создаём пользователя и отвечаем на вопросы, которые будут задаваться
adduser имя_пользователя
```

Задаём пароль для созданного пользователя:
```sh
passwd имя_пользователя
```

Тепереь нужно добавить созданного пользователя в *sudo*.
Можно об этом почитать [здесь](https://www.8host.com/blog/redaktirovanie-fajla-sudoers-v-ubuntu-i-centos/).
Лучше это делать через **visudo**. Для открытия файла */etc/sudoers* вводим:
```sh
sudo visudo
```
Нужно добавить строку `имя_пользователя ALL=(ALL) ALL` и сохраниться.

Если доступна команда `adduser`, то можно сделать так:
```sh
adduser имя_пользователя sudo
```
На крайний случай можно сделать так:
```sh
echo 'имя_пользователя ALL=(ALL) ALL' >> /etc/sudoers
```

Теперь нужно отключить авторизацию от *root*-а:
```sh
nano /etc/ssh/sshd_config
```
Нужно заменить строку `PermitRootLogin yes` на `PermitRootLogin no`.

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
Если нужно полностью отключить вход по паролю и оставить только вход по *ssh-ключу*,
то:
```sh
# открываем файл
nano /etc/ssh/sshd_config
```
меняем строку `PasswordAuthentication yes` на `PasswordAuthentication no` и рестартуем ssh-сервис:
```sh
systemctl restart sshd
```
#### Настройка *firewall*-а
В данном примере показывается настройка *firewall*-а **ufw**.
Если он не установлен (для проверки введите например `sudo ufw status`), то устанавливаем, но сначала лучше обновить пакеты:
```sh
sudo apt update && sudo apt upgrade
```
и устанавливаем сам *firewall*:
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
