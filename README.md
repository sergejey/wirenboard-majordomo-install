# Установка MajorDoMo на Wirenboard

[Wirenboard](https://wirenboard.com/ru/) это модульный контроллер на базе ОС Linux, который может использоваться как отдельный модуль автоматизации, при этом он достаточно мощный и при желании прямо на нём можно развернуть платформу MajorDoMo. Для серьёзных проектов такой вариант не очень подходит в виду ограничений производительности контроллера, но, в том случае, когда требуется минимальное по оборудованию решение, но при этом хочется иметь удобный пользовательский интерфейс и простые средства настройки, то предолженный вариант может быть вполне уместен. Настроенная подобным образом система тестировалась в работе несколько месяцев и показала себя вполне надёжным решением без каких-либо серьёзных проблем со стабильностью и производительностью.

В данном примере использовался контроллер [Wiren Board 6](https://wirenboard.com/wiki/index.php/Wiren_Board_6).

## БАЗОВАЯ УСТАНОВКА

Подключаемся по SSH к Wirenboard (по-умолчанию root / wirenboard)

Обновляем резпозитории
```
apt-get update
```

Устанавливаем php и дополнительные пакеты
```
apt-get install php-fpm php-common mysql-client php-pear php-mysql php-curl php-gd php-bcmath php-imagick php-imap php-mcrypt php-pspell php-recode php-tidy php-xml php-json php-mbstring
```

Устанавливаем базу данных
```
apt-get install mariadb-server
```

Настраиваем имя пользователя и пароль базы данных
```
mysql_secure_installation
```
(там главное установить root-пароль для базы данных -- мы его будем использовать в дальнейшем, для примера возьмём 'rootpsw')

Перейдём в папку
```
cd /mnt/data
```

Скачаем исходный код majordomo

```
wget https://github.com/sergejey/majordomo/archive/master.tar.gz
```

(или) 
```
wget https://github.com/sergejey/majordomo/archive/alpha.tar.gz
```
(если хотите самую свежую версию для разработчиков)

распаковываем архив
```
tar xzvf master.tar.gz
```
(или) 
```
tar xzvf alpha.tar.gz
```

переименовываем папку в majoromo
```
mv majordomo-master/ majordomo
```

(или )

```
mv majordomo-alpha/ majordomo
```

разрешаем запись в папку
```
chmod -Rf 0777 majordomo/
```

заходим в папку
```
cd majordomo/
```

(последующие команды предполагают нахождение в папке /mnt/data/majordomo)

переименовываем пример конфига в обычный конфиг

```
mv config.php.sample config.php
```

редактируем конфиг

```
nano config.php
```

в конфиге надо установить пароль DB_PASSWORD установленый нами root-пароль для базы данных

в опции SERVER_ROOT прописываем путь /mnt/data/majordomo

в опции BASE_URL меняем порт с :80 на :82

выходим из редактирования нажатием Ctrl+X с сохранением изменений

## НАСТРОЙКА БАЗЫ ДАННЫХ

Останавливаем сервис базы данных
```
service mysql stop
```

Переносим каталог базы данных
```
mv /var/lib/mysql /mnt/data/var/lib/mysql/
```

Создаём ссылку с нового на старое место
```
ln -s /mnt/data/var/lib/mysql /var/lib/mysql
```

Запускаем сервис базы данных
```
service mysql start
```

Запускаем консоль базы данных
```
mysql -u root -p
```
(потребуется ввести root-пароль)

Выполняем следующие команды:
```
CREATE DATABASE db_terminal;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'rootpsw';
FLUSH PRIVILEGES;
exit
```

Запускаем импорт дампа базы данных:
```
mysql -u root --password=rootpsw db_terminal<db_terminal.sql
```
(вместо rootpsw используйте свой root-пароль)

## НАСТРАИВАЕМ NGINX

Заходим в папку
```
cd /etc/nginx/sites-enabled/
```

скачиваем файл настроек
```
wget https://github.com/sergejey/wirenboard-majordomo-install/raw/master/majordomo_nginx
```

Перезапускаем nginx
```
service nginx restart
```

## ЗАПУСК ОСНОВНОГО ЦИКЛА

Заходим в папку
```
cd /etc/init.d/
```

скачиваем файл инициализации
```
wget https://github.com/sergejey/wirenboard-majordomo-install/raw/master/majordomo_init
```

ставим атрибуты
```
chmod 0755 majordomo_init
```

ставим автозагрузку
```
update-rc.d majordomo_init defaults
```

запускаем цикл
```
/etc/init.d/majordomo_init start
```

## ВЕБ-ИНТЕРФЕЙС

Переходим в веб-интерфейс http://IP:82/admin.php

Через раздел System -> Plugins Market ставим модуль Wirenboard

Прописываем ему следующие настройки:
![](https://c2n.me/40LENqd.png)

Дополнительно можно настроить часовой пояс и язык.

Раздел Settings -> General settings
