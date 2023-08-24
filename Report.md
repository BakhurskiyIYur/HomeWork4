# Домашнее задание №4

Описание/Пошаговая инструкция выполнения домашнего задания:
yc compute instance create  --name vm-ubuntu  --hostname vm-ubuntu  --zone ru-central1-a  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4  --memory 4G  --cores 2  --zone ru-central1-a  --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts,auto-delete=true  --metadata-from-file ssh-keys=/Users/GPN-A/.ssh/filessh.txt
1 Создайте новый кластер PostgresSQL 14
sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-14
pg_lsclusters
2 Зайдите в созданный кластер под пользователем postgres
sudo -u postgres psql
3 Создайте новую базу данных testdb
CREATE DATABASE testdb;
\l – список всех БД
4 Зайдите в созданную базу данных под пользователем postgres
\c testdb
5 Создайте новую схему testnm
select current_database();
CREATE SCHEMA testnm;
select schema_name from information_schema.schemata;
6 Создайте новую таблицу t1 с одной колонкой c1 типа integer
CREATE TABLE t1(c1 integer);
7 Вставьте строку со значением c1=1
INSERT INTO t1 values(1);
8 Создайте новую роль readonly
CREATE role readonly;
\du
9 Дайте новой роли право на подключение к базе данных testdb
grant connect on DATABASE testdb TO readonly;
10 Дайте новой роли право на использование схемы testnm 
grant usage on SCHEMA testnm to readonly;
11 Дайте новой роли право на select для всех таблиц схемы testnm 
grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
12 Создайте пользователя testread с паролем test123 
CREATE USER testread with password 'test123';
\du – список всех ролей/пользователей
13 Дайте роль readonly пользователю testread 
grant readonly TO testread; 
\du
14 Зайдите под пользователем testread в базу данных testdb
psql -h 127.0.0.1 -U testread -d testdb -W
15 Сделайте select * from t1;
16 Получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
Не получилось прочитать таблицу t1, нет прав доступа. 
17 Напишите что именно произошло в тексте домашнего задания
Создали таблицу t1 в базе данных testdb, и у пользователя testread  есть доступ к базе testdb из-за роли readonly. Казалось бы что пользователь testread должен иметь доступ на чтение к ней, а доступа нет.  В задании сказано было создать таблицу t1, но не указали в какой схеме.
18 У вас есть идеи почему? ведь права то дали?
Потому что права на testnm для роли readonly дали, а права на public для роли readonly не дали. А таблица создана как раз в  схеме public.
19 Посмотрите на список таблиц
\dt – список всех таблиц в текущей базе данных
20 Подсказка в шпаргалке под пунктом 20
21 А почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
Потому что при создании таблицы t1 не была указана явно схема testnm, в которой создать. Поэтому таблица создалась по умолчанию в схеме public. 
Прав на public для роли readonly не давали.
22 Вернитесь в базу данных testdb под пользователем postgres
\c testdb postgres
23 Удалите таблицу t1
DROP TABLE t1;
24 Создайте ее заново но уже с явным указанием имени схемы testnm
CREATE TABLE testnm.t1(c1 integer);
25 Вставьте строку со значением c1=1
INSERT INTO testnm.t1 values(1);
26 Зайдите под пользователем testread в базу данных testdb
psql -h 127.0.0.1 -U testread -d testdb -W
27 Сделайте select * from testnm.t1;
28 Получилось?
Нет, не получилось, нет прав доступа.
29 Есть идеи почему? если нет - смотрите шпаргалку
Потому что выполненная ранее команда «grant SELECT on all TABLEs in SCHEMA testnm TO readonly;» была актуальна для существующих таблиц на тот момент. 
С момента ее выполнения таблица пересоздавалась, и теперь нужно повторно выполнить команду «grant SELECT on all TABLEs in SCHEMA testnm TO readonly;»
30 Как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
\c testdb postgres; 
ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
\c testdb testread;
31 Сделайте select * from testnm.t1;
select * from testnm.t1;
32 Получилось?
Снова не получилось, нет прав доступа.
33 Есть идеи почему? если нет - смотрите шпаргалку
Потому что команда «ALTER default…» будет действовать для новых таблиц, а команда «grant SELECT on all TABLEs in SCHEMA testnm TO readonly» отработала только для существующих таблиц на тот момент времени. Нужно выполнить снова  команду «grant SELECT…» или пересоздать таблицу.
31 Сделайте select * from testnm.t1;
32 Получилось?
Да.
33 Ура!
34 Теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
35 А как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
Таблица создалась в схеме public, по умолчанию, так как не указал явно схему при создании таблицы. 
Каждый пользователь может по умолчанию создавать объекты в схеме public в базе данных, если у него есть право на подключение к этой базе данных.
А право на подключение к базе testdb у пользователя testread есть.
36 Есть идеи как убрать эти права? если нет - смотрите шпаргалку
Нужно отменить права на создание таблиц в схеме public для пользователя testread.
\c testdb postgres; 
REVOKE CREATE on SCHEMA public FROM public; 
REVOKE ALL on DATABASE testdb FROM public; 
\c testdb testread;
37 Если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
Под суперпользователем postgres запретил создавать таблицы в схеме public в базе testdb.
Команда «REVOKE CREATE on SCHEMA public FROM public; » отменила права на создание чего либо в схеме public  всем пользователям.
Команда « REVOKE ALL on DATABASE testdb FROM public; » отменила всем все права по отношению к базе данных  testdb  .
38 Теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
39 Расскажите что получилось и почему
Теперь создать таблицу в БД testdb не получилось. Права на создание таблиц в схеме  public отменены. 

