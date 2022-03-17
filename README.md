# Howto work with Postgresql

Notes
-----

```
sudo apt-get install postgresql
sudo apt-get install postgresql-client
sudo apt-get install postgresql-contrib

service postgresql status
```

### Команды сервиса
service postgresql

sudo -i -u postgres


Если необходимо произвести сразу несколько действий от имени другого пользователя, то для этого можно запустить новую оболочку внутри текущей (говорят что мы стартуем новую сессию):

sudo -i
id

uid=0(root) gid=0(root) groups=0(root)
Главное — не забыть переключиться обратно после завершения необходимых манипуляций. Для этого наберите exit.

Запускаем консоль Postgresql:
psql


Вывод списка баз данных:
psql=# \l
выйти из режима просмотра: q
Выйти из консоли: \q


Создание базы данных:
createdb bot_users
Вывод списка баз данных:
psql=# \l

Удаление базы:
dropdb bot_users

Изменим параль суперпользователя Postgresql
psql
# просмотр списка пользователей
psql=# \du
psql=# ALTER USER postgres WITH PASSWORD 'qwerty';

# Создание пользователя
psql=# CREATE USER username WITH PASSWORD 'password';

Предоставление прав новому пользователю:
psql=# ALTER USER username WITH SUPERUSER;

Удаление пользователя:
psql=# DROP USER usernameж

# выйти из учетной записи
exit

#Вызов справки
man psql


Установка pgAdmin

https://www.pgadmin.org/download/pgadmin-4-apt/

После установки вводим параметры нового сервера
Server name: localhost
Host name/address: 127.0.0.1

Работа с серью. Открытые порты
$ ss -ltnp
vim /etc/postgresql/12/main/postgresql.conf


data_directory # хранение всех файлов баз данных
hba_file # определяет параметры подключения к базе данных
#listen_addresses = 'localhost' # выбор способа работы в сети 
(localhost - локальная сеть)
(* - любая сеть)

# RESOURCE USAGE
#-------------
shared_pages = 128MB


# Перезагрузка файла конфигурации
systemctl reload postgresql

# Проверка состояния сервисам СУБД
systemctl status postgresql

# Лог системы
vim /var/log/syslog
ps aux | grep postgres

# Узнать размер директории (папки)
du -hs /var/lib/postgresql/12/main/

# Поменять пользователя
su postgres
# Запустить psql


sudo -i -u postgres
psql


Вывод списка баз данных:
psql=# \l

# создать раздел
CREATE DATABASE unixway1;
CREATE DATABASE unixway1;

# создать пользователей
CREATE USER unixway1user WITH ENCRYPTED PASSWORD 'password1';
GRANT ALL PRIVILEGES ON DATABASE unixway1 TO unixway1user;

# Поменять владельца раздела
ALTER DATABASE unixway1 OWNER TO unixway1user;


# создать пользователей
CREATE USER unixway2user WITH ENCRYPTED PASSWORD 'password2';
GRANT ALL PRIVILEGES ON DATABASE unixway2 TO unixway2user;
# Поменять владельца раздела
ALTER DATABASE unixway2 OWNER TO unixway2user;

# убрать доступ к базам данных извне сервера
REVOKE CONNECT ON DATABASE unixway1 FROM PUBLIC;
REVOKE CONNECT ON DATABASE unixway2 FROM PUBLIC;

#Подключиться к базе данных с помощью локального пользователя
psql -h 127.0.0.1 -p 5432 -U unixway1user unixway1

Добавить строку в файл pg_hba.conf
host replication all 192.168.0.103/24 md5

Обновить конфигурацию postgres:
systemctl reload postgresql

Переподключиться с помощью другой базе данных
unixway2=> \c unixway1


# Создание таблицы
CREATE TABLE users(user_pid SERIAL PRIMARY KEY, user_name TEXT NOT NULL, user_email TEXT NOT NULL, user_balance INTEGER NOT NULL);
unixway2=> \d

Просмотр списка талиц:
unixway1=> \d users
unixway1=> \c unixway1

#Добавим данные:
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Jerry Kinn','jkinn@gmail.com',1000);
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Adam Smith','asmith@gmail.com',1000);
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Billy Jonnes','bjonnes@gmail.com',1000);

# Печать содержимого таблицы:
SELECT * FROM users;

unixway1=> \d 

# Удалить талицу
unixway1=> DROP TABLE users;

# Активировать дополнение - использование типа данных uuid
unixway1=> CREATE EXTENSION IF NOT EXITS "uuid-ossp";
# Активировать расширение нужно только один раз для каждого раздела, где он будет использоваться

# Создадим таблицу, используя UUID
CREATE TABLE users(user_pid SERIAL PRIMARY KEY, user_id UUID NOT NULL DEFAULT uuid_generate_v4(), user_name TEXT NOT NULL, user_email TEXT NOT NULL, user_balance INTEGER NOT NULL, user_registration TIMESTAMP NOT NULL DEFAULT now());

# Демонстрация работы функции uuid_generate_v4()
SELECT uuid_generate_v4();

#Добавим данные:
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Jerry Kinn','jkinn@gmail.com',1000);
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Adam Smith','asmith@gmail.com',1000);
INSERT INTO users(user_name, user_email,user_balance) VALUES ('Billy Jonnes','bjonnes@gmail.com',1000);

# Печать содержимого таблицы:
SELECT * FROM users;

# Попытка добавить неполные данные:
INSERT INTO users(user_name,user_email) VALUES ('Martha Longdale','mlngdale@gmail.com');

# Модификация базы данных
ALTER TABLE users ALTER COLUMN user_balance SET DEFAULT 0;


# Модификация базы данных
ALTER TABLE users ADD CHECK(user_balance >= 0);

# Модификация базы данных
ALTER TABLE users ADD CHECK(length(user_name) <= 100);
ALTER TABLE users ADD CHECK(length(user_email) <= 100);

# Попытка добавить неполные данные:
INSERT INTO users(user_name,user_email) VALUES ('Niels Bourre','nbourre@gmail.com', -100);

# Модификация базы данных
ALTER TABLE users ADD COLUMN user_enabled BOOLEAN NOT NULL DEFAULT true;


# Модификация базы данных. Удалим колонку user_enabled
ALTER TABLE users DROP COLUMN user_enabled;

# Обновить данные пользователя
UPDATE users SET user_balance = user_balance + 100 WHERE user_pid = 2;
UPDATE users SET user_balance = user_balance + 100 WHERE user_id = '691967ea-5648-4529-8d70-1a0843683c3d';
UPDATE users SET user_balance = user_balance + 10;
UPDATE users SET user_balance = user_balance - 100;
UPDATE users SET user_balance = user_balance + 100 WHERE user_balance < 100;
UPDATE users SET user_balance = 0;

# Удалить все строки таблицы users
DELETE FROM users;
