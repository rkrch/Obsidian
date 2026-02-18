```
sudo apt-get update
sudo apt-get upgrade
sudo apt install wget git gcc gcc-c++ svn cmake make automake autoconf pkgconfig graphviz
sudo apt -y install curl libnewt-dev libssl-dev libncurses5-dev subversion libsqlite3-dev build-essential libjansson-dev libxml2-dev uuid-dev
sudo apt-get install codec2
sudo apt-get install libedit-dev
sudo apt-get install libsrtp2-dev
sudo apt-get install unixodbc-dev
sudo apt-get install unixodbc
sudo ufw allow 5060/tcp
sudo ufw allow 5060/udp
sudo ufw allow 5061/tcp
sudo ufw allow 5061/udp
sudo ufw allow 4569/udp
sudo ufw allow 10000:20000/udp
sudo ufw allow 22
sudo ufw enable

wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-18-current.tar.gz
tar xvf asterisk-18-current.tar.gz
cd asterisk-18*/
contrib/scripts/get_mp3_source.sh
make distclean
sudo ./configure

sudo make menuselect
```


Необходимые компоненты:
- Add-ons: format_mp3;
- Call Detail Recording: убрать ~~cdr_radius~~, cdr_sqlite3_custom, ~~cdr_tds~~;
- Channel Event Logging: убрать ~~cel_radius~~, cel_sqlite3_custom, ~~cel_tds~~;
- Channel Drivers: оставить только chan_iax2, chan_pjsip, chan_rtp;
- Codec Translators: добавить codec_opus;
- Resource Modules: убрать res_agi, все пункты с res_ari, res_fax, res_phoneprov, res_smdi (эти модули не нужны в данной установке и вызывают появление ошибок при запуске);
- Compiler Flags: LOW_MEMORY, G711_NEW_ALGORITHM, G711_REDUCED_BRANCHING;
- Core Sound Packages: только RU-WAV.
После этого выйдите, нажав _**Save & Exit**_.

```
sudo make
sudo make install

sudo make samples
sudo make config

```
Необходимо раскомментировать (удалить ;) и редактировать следующие пункты:

```shell
runuser = asterisk 
rungroup = asterisk  
defaultlanguage = ru
```

Также откройте файл _**/etc/default/asterisk**_ и раскомментируйте строки ниже

```shell
AST_USER="asterisk"
AST_GROUP="asterisk"
```

Создайте группу и служебного пользователя asterisk, от имени которого будет работать служба Asterisk (этот пользователь не сможет осуществлять вход в систему):

```shell
sudo groupadd asterisk
sudo useradd -r -d /var/lib/asterisk -g asterisk asterisk
sudo usermod -aG audio,dialout asterisk
```

Смените владельца следующих директорий на пользователя _asterisk_ для предоставления ему полного доступа:

```shell
sudo chown -R asterisk.asterisk /etc/asterisk
sudo chown -R asterisk.asterisk /var/{lib,log,spool}/asterisk
sudo chown -R asterisk.asterisk /usr/lib/asterisk
sudo chmod -R 750 /var/{lib,log,run,spool}/asterisk /usr/lib/asterisk /etc/asterisk
```

Запуск Asterisk для проверки корректности установки и первоначальной конфигурации:

```shell
sudo asterisk -c
```

Если Asterisk успешно запустится, в конце вывода служебных сообщений появится зеленая надпись Asterisk Ready и приглашение командной строки Asterisk _CLI> (возможно появление предупреждений и ошибок — это обусловлено особенностями сгенерированной конфигурации по умолчанию, которая предполагает использование некоторых не сконфигурированных в текущей установке Asterisk модулей. На работоспособность в целом не влияет)._

Выход обратно в bash: _**Ctrl+C**_.

Запретите загрузку вызывающих ошибки модулей, которые не понадобятся в текущей работе:

```shell
sudo vi /etc/asterisk/modules.conf
```

вставить после строки _autoload=yes_:

```shell
noload => pbx_dundi
noload => res_config_ldap
noload => res_pjsip_phoneprov_provider
```

Если на предыдущем шаге Asterisk успешно запустился, можно запустить Asterisk как фоновую службу с автозапуском на старте ОС:

```shell
sudo systemctl restart asterisk && sudo systemctl enable asterisk
```

Проверьте, что Asterisk корректно запущен в виде фоновой службы:

Если служба корректно запущена и работает, в выводе должно быть указано active (running) (Рисунок 2.7).

```shell
systemctl status asterisk
```

[![Выбор сетевого моста в Virtual Box](https://github.com/my-psvhs/Asterisk_26/raw/main/image/running.png)](https://github.com/my-psvhs/Asterisk_26/blob/main/image/running.png)

_Рис 2.7. Служба asterisk успешно запущена_

Если получено иное, перезапустите службу и еще раз проверьте статус:

```shell
sudo systemctl restart asterisk
```

```shell
sudo systemctl status asterisk
```

Проверьте возможность подключения к интерфейсу CLI Asterisk:

```shell
sudo asterisk -vvvr
```

Подключаться к Asterisk Вам потребуется позже при отладке конфигурации, пока что выйдите обратно в bash.

Для установки кодека OPUS необходимо сделать следующие действия

Скачать кодек с помощью wget

```shell
sudo wget https://downloads.digium.com/pub/telephony/codec_opus/asterisk-18.0/x86-64/codec_opus-18.0_1.3.0-x86_64.tar.gz
```

Далее необходимо загрузить этот модуль в asterisk, отредактировав файл _/etc/asterisk/modules.conf_ Добавим строку

```shell
load => codec_opus.so
```

Перезапустим службу asterisk. И проверим успешную загрузку кодека OPUS. Снова подключимся к интерфейсу CLI Asterisk и введем команду для того, чтобы посмотреть какие кодеки он поддерживает

```shell
core show codecs
```

[![Выбор сетевого моста в Virtual Box](https://github.com/my-psvhs/Asterisk_26/raw/main/image/OPUS.png)](https://github.com/my-psvhs/Asterisk_26/blob/main/image/OPUS.png)
_Рис 2.8. Кодек OPUS подключен_

[дальше тут короч](https://github.com/my-psvhs/Asterisk_26/blob/main/Asterisk.md#23-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D1%8F-asterisk-%D0%B4%D0%BB%D1%8F-%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B8-%D0%B2%D1%8B%D0%B7%D0%BE%D0%B2%D0%BE%D0%B2)
