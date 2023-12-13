# HOMEWORK #7. Репликация.

### 1. Физическая репликация.
#### 1.1. Настроить физическую репликации между двумя кластерами базы данных. ####

> Подготовил два кластера [main (5432 порт), main2 (5433 порт)], wal_level = replica

```bash
varefyev@varefyev-VirtualBox:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
12  main    5432 online postgres /var/lib/postgresql/12/main  /var/log/postgresql/postgresql-12-main.log
12  main2   5433 online postgres /var/lib/postgresql/12/main2 /var/log/postgresql/postgresql-12-main2.log
```

#### 1.2. Репликация должна работать использую "слот репликации" ####

> Используем утилиту pg_basebackup, но перед использованием - очищаем кластер main2, чтобы сделать базовую резервную копию.

R - включает создание конфигурации восстановления;

D - указывает каталог данных;

С - создает слот репликации;

S - имя для слота репликации.

```bash
sudo rm -rf /var/lib/postgresql/12/main2

sudo -u postgres pg_basebackup -R -D /var/lib/postgresql/12/main2 -S baseslot
```
> Убедимся в том, что слот репликации создан:
```sql 
otus=# SELECT * FROM pg_replication_slots \gx
-[ RECORD 1 ]-------+----------
slot_name           | pg_slot
plugin              |
slot_type           | physical
datoid              |
database            |
temporary           | f
active              | t
active_pid          | 28725
xmin                |
catalog_xmin        |
restart_lsn         | 0/A014350
confirmed_flush_lsn |
```

#### 1.3. Реплика должна отставать от мастера на 5 минут ####

```sql
alter system set recovery_min_apply_delay = 300000; -- 5 минут
SELECT pg_reload_conf(); -- применить изменения, если не получилось, то перезапустить кластер. 
```
> Перезапуск кластера:
```bash 
sudo pg_ctlcluster 12 main2 restart # перезапуск кластера №2
```
#### Практика. ####
> Требуется создать БД на master и slave (otus - для примера).

```bash
sudo -u postgres psql -p 5432 # подключаемся к кластеру №1 (main) на порт 5432
sudo -u postgres psql -p 5433 # подключаемся к кластеру №2 (main2) на порт 5433
```
```sql
postgres=# create database otus; -- создали БД otus
CREATE DATABASE

postgres=# \c otus -- подключились к БД otus

otus=# create table students as select generate_series(1,10) as id, md5(random()::text)::char(10) as fio; -- создали табличку и заполнили ее тестовыми данными

otus=# insert into students values (11, '123test123'); -- добавим новую запись, каждый шаг можно чекать в отдельном терминале, чтобы убедиться в том, что все работает.
INSERT 0 1
otus=# select * from students;
 id |    fio     
----+------------
  1 | ec4c98d563
  2 | 05924d596a
  3 | 80b926b6e2
  4 | 86d6a58dc4
  5 | 7bad073f74
  6 | 1d3afad138
  7 | 146cd6d731
  8 | 2f967fde13
  9 | 1dd331f549
 10 | 6e1367b9f0
 11 | 123test123
(11 rows)
```

> После 5 минут на slave отобразилась добавленная запись.


### 2. Логическая репликация.
> Подготовил два кластера [main (5432 порт), main3 (5434 порт)], wal_level = logical

```bash
varefyev@varefyev-VirtualBox:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
12  main    5432 online postgres /var/lib/postgresql/12/main  /var/log/postgresql/postgresql-12-main.log
12  main3   5434 online postgres /var/lib/postgresql/12/main3 /var/log/postgresql/postgresql-12-main3.log
```

#### 2.1. Создать на первом кластере базу данных, таблицу и наполнить ее данными (на ваше усмотрение). ####

```sql 
postgres=# create database replica;
CREATE DATABASE
postgres=# \c replica
You are now connected to database "replica" as user "postgres".
replica=# create table students as select generate_series(1,10) as id, md5(random()::text)::char(10) as fio;

```
> Сменим в конфигурационном файле wal_level {replica -> logical}.

``` sql
replica=# alter system set wal_level = logical; 
ALTER SYSTEM
replica=# SELECT pg_reload_conf(); -- если не помогло, то перезагружаем кластер (команда была дана выше).
```

#### 2.2. На нем же создать публикацию этой таблицы ####
```sql
replica=# create publication test_pub for table students;
CREATE PUBLICATION

```

#### 2.3. На новом кластере подписаться на эту публикацию ####
```sql
postgres=# create database replica1;
CREATE DATABASE

replica1=# create table students as select generate_series(1,10) as id, md5(random()::text)::char(10) as fio where 0=1;

replica1=# create subscription test_sub2 connection 'host=localhost port=5432 user=postgres password=24051995 dbname=replica' publication test_pub with (copy_data = true);
NOTICE:  created replication slot "test_sub2" on publisher
CREATE SUBSCRIPTION
```
#### 2.3. Убедиться что она среплицировалась. Добавить записи в эту таблицу на основном сервере и убедиться, что они видны на логической реплике. ####


> Проверим, создалась ли публикация и подсписчик:

![скрин №1](![Alt text](image-6.png))

> Выполним проверку, добавим в БД replica одну запись, но перед добавлением - выполним select по таблице students в БД replica1 (убедимся, что записей столько - сколько и в БД replica). Добавим запись в БД replica и выполним повторный select. Убедимся в том, что логическая репликация работает.

![скрин №2](![Alt text](image-7.png))
