Повторите шаги Алексея:

Создайте виртуальную машину с Ubuntu 20.04 и установите PostgreSQL 15 или выше.
Создайте таблицу с данными о перевозках.
Добавьте внешний диск к виртуальной машине и перенесите туда базу данных.
Настройте PostgreSQL для работы с новым диском.
Проверьте, что данные сохранились и доступны.


1.
В большой фирме, занимающейся трансатлантическими перевозками,  БД будем выносить на внешний трердотельный носитель, но приступим. Яндекс облако сейчас не доступно, будем делать на простом ноутбуке с докером и убунтой.

готовим внешнее хранилище
```
Command (m for help): p

Disk /dev/sda: 14,4 GiB, 15462301696 bytes, 30199808 sectors
Disk model: SD/MMC          
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xb1c0d181

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-30199807, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-30199807, default 30199807): 

Created a new partition 1 of type 'Linux' and of size 14,4 GiB.

Command (m for help): p
Disk /dev/sda: 14,4 GiB, 15462301696 bytes, 30199808 sectors
Disk model: SD/MMC          
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xb1c0d181

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1        2048 30199807 30197760 14,4G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@tpt480:/home/serge# mkfs.ext4  /dev/sda1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 3774720 4k blocks and 944704 inodes
Filesystem UUID: 4806118c-922a-4283-91a0-223690756419
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done   

```
монтируем в отдельный раздел
```
root@tpt480:/home/serge# mount /dev/sda1 /mnt/sd
mount: /mnt/sd: mount point does not exist.
       dmesg(1) may have more information after failed mount system call.
root@tpt480:/home/serge# mkdir /mnt/sd
root@tpt480:/home/serge# mount /dev/sda1 /mnt/sd
```
создаем на локальном диске папку для ПГ15, запускаем ПГ15

```
mkdir /opt/postgres15
docker run --rm -d  --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6001:5432   -v /opt/postgres15:/var/lib/postgresql/data postgres:15
```

подключаемся клиентом и заливаем данные логистики
```
root@tpt480:/home/serge# psql  -h localhost -p 6001 -U postgres 
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 15.13 (Debian 15.13-1.pgdg130+1))
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "localhost" (address "127.0.0.1") at port "6001".

```
postgres=# create table shipments(id serial, product_name text, quantity int, destination text);
CREATE TABLE
postgres=# insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('bananas', 1500, 'Asia');
insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');
insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
insert into shipments(product_name, quantity, destination) values('coffee', 700, 'Canada');
insert into shipments(product_name, quantity, destination) values('coffee', 300, 'Japan');
insert into shipments(product_name, quantity, destination) values('sugar', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('sugar', 800, 'Asia');
insert into shipments(product_name, quantity, destination) values('sugar', 600, 'Africa');
insert into shipments(product_name, quantity, destination) values('sugar', 400, 'USA');
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
```
выходим и останвливаем контейнер.  
Переносим папку с данными с локального на внешний диск, проверяем размер

```

root@tpt480:/home/serge# mv -v /opt/postgres15 /mnt/sd/
created directory '/mnt/sd/postgres15'
created directory '/mnt/sd/postgres15/pg_wal'
created directory '/mnt/sd/postgres15/pg_wal/archive_status'
copied '/opt/postgres15/pg_wal/000000010000000000000001' -> '/mnt/sd/postgres15/pg_wal/000000010000000000000001'
created directory '/mnt/sd/postgres15/global'
copied '/opt/postgres15/global/pg_control' -> '/mnt/sd/postgres15/global/pg_control'
copied '/opt/postgres15/global/1262' -> '/mnt/sd/postgres15/global/1262'
copied '/opt/postgres15/global/2964' -> '/mnt/sd/postgres15/global/2964'
...
copied '/opt/postgres15/base/4/3429' -> '/mnt/sd/postgres15/base/4/3429'
copied '/opt/postgres15/base/4/2618' -> '/mnt/sd/postgres15/base/4/2618'
copied '/opt/postgres15/base/4/2620' -> '/mnt/sd/postgres15/base/4/2620'
..
removed '/opt/postgres15/postgresql.auto.conf'
removed '/opt/postgres15/pg_hba.conf'
removed '/opt/postgres15/pg_ident.conf'
removed '/opt/postgres15/postmaster.opts'
removed '/opt/postgres15/postmaster.pid'
removed directory '/opt/postgres15'


root@tpt480:/home/serge# ls -l /mnt/sd/
total 24
drwx------  2 root    root 16384 окт 14 14:15 lost+found
drwx------ 19 polkitd root  4096 окт 14 14:37 postgres
drwx------ 19 polkitd root  4096 окт 14 14:44 postgres15
root@tpt480:/home/serge# du -sm /mnt/sd/*
1	/mnt/sd/lost+found
39	/mnt/sd/postgres
39	/mnt/sd/postgres15

```
запускаем ПГ15 в докере с новым volume данными.
подключаемся локальным клиентом psql, проверяем данные. все на месте после переноса.
```
root@tpt480:/home/serge# docker run --rm -d  --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6001:5432   -v /mnt/sd/postgres15:/var/lib/postgresql/data postgres:15
bbe321f5a4a0c0946a7c117514583acfb9f9a722b5a26b380a6e3dc7b2fce8be
root@tpt480:/home/serge# psql  -h localhost -p 6001 -U postgres 
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 15.13 (Debian 15.13-1.pgdg130+1))
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "localhost" (address "127.0.0.1") at port "6001".
postgres=# \dt
           List of relations
 Schema |   Name    | Type  |  Owner   
--------+-----------+-------+----------
 public | logistics | table | postgres
 public | shipments | table | postgres
(2 rows)

postgres=# select * from shipments;
 id | product_name | quantity | destination 
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | bananas      |     1500 | Asia
  3 | bananas      |     2000 | Africa
  4 | coffee       |      500 | USA
  5 | coffee       |      700 | Canada
  6 | coffee       |      300 | Japan
  7 | sugar        |     1000 | Europe
  8 | sugar        |      800 | Asia
  9 | sugar        |      600 | Africa
 10 | sugar        |      400 | USA
(10 rows)

```
добавление данных в табличку также прошло успешно.

```
postgres=# insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
INSERT 0 1
postgres=# select * from shipments;
 id | product_name | quantity | destination 
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | bananas      |     1500 | Asia
  3 | bananas      |     2000 | Africa
  4 | coffee       |      500 | USA
  5 | coffee       |      700 | Canada
  6 | coffee       |      300 | Japan
  7 | sugar        |     1000 | Europe
  8 | sugar        |      800 | Asia
  9 | sugar        |      600 | Africa
 10 | sugar        |      400 | USA
 34 | coffee       |      500 | USA
(11 rows)
```

