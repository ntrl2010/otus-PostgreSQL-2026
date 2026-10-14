докер установлен в убунте.

```
root@tpt480:/home/serge# docker -v
Docker version 24.0.7, build 24.0.7-0ubuntu4
root@tpt480:/home/serge# uname -a
Linux tpt480 6.8.0-36-generic #36-Ubuntu SMP PREEMPT_DYNAMIC Mon Jun 10 10:49:14 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```
создаем отдельную сеть pg-net для экспериментов 
```
root@tpt480:/home/serge# docker network create pg-net
1f65d9b9898c706ded3dad81759f017351e4324f3b4642f79bb917dc5e661790
```

создаем отдельный каталог для данных
```
root@tpt480:/home/serge# mkdir /opt/postgres

запускаем постгрес с "пробросом" каталога
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 5432:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14
Unable to find image 'postgres:14' locally
14: Pulling from library/postgres
ecc510c1e359: Pull complete 
...
73ab60ca07a1: Pull complete 
Digest: sha256:14bfab572eec6abf65892e1db7c3ba8d41b2a2855c1b143664907d6e146eb6e1
Status: Downloaded newer image for postgres:14
a616a13cc29b071849e0f317ba339e16349b607f65b864970270920c2c38398f
docker: Error response from daemon: driver failed programming external connectivity on endpoint pg-server (f8494334c0e5e186134e0b8777e1c68dbcc91a5b60fc7cc622ecee6c3d1775ac): Error starting userland proxy: listen tcp4 0.0.0.0:5432: bind: address already in use.
```
ошибка изза занятого порта по умолчанию

```
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 5433:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14

повторная попытка, ошибка с ссуществующим именем, удаляем хвосты предыдущих попыток запуска.

docker: Error response from daemon: Conflict. The container name "/pg-server" is already in use by container "a616a13cc29b071849e0f317ba339e16349b607f65b864970270920c2c38398f". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
```

проверка занятых портов

```
root@tpt480:/home/serge# ss -ltpn
State   Recv-Q  Send-Q    Local Address:Port      Peer Address:Port  Process                                                      
LISTEN  0       4096            0.0.0.0:8080           0.0.0.0:*      users:(("docker-proxy",pid=4361,fd=4))                      
LISTEN  0       5               0.0.0.0:4330           0.0.0.0:*      users:(("pmlogger",pid=3230,fd=7))                          
LISTEN  0       4096         127.0.0.54:53             0.0.0.0:*      users:(("systemd-resolve",pid=1238,fd=17))                  
LISTEN  0       4096            0.0.0.0:111            0.0.0.0:*      users:(("rpcbind",pid=1224,fd=4),("systemd",pid=1,fd=189))  
LISTEN  0       5               0.0.0.0:44321          0.0.0.0:*      users:(("pmcd",pid=2534,fd=0))                              
LISTEN  0       128             0.0.0.0:44323          0.0.0.0:*      users:(("pmproxy",pid=2559,fd=15))                          
LISTEN  0       128             0.0.0.0:44322          0.0.0.0:*      users:(("pmproxy",pid=2559,fd=13))                          
LISTEN  0       4096          127.0.0.1:33529          0.0.0.0:*      users:(("containerd",pid=2057,fd=8))                        
LISTEN  0       4096          127.0.0.1:631            0.0.0.0:*      users:(("cupsd",pid=2038,fd=8))                             
LISTEN  0       200           127.0.0.1:5435           0.0.0.0:*      users:(("postgres",pid=2146,fd=6))                          
LISTEN  0       200           127.0.0.1:5434           0.0.0.0:*      users:(("postgres",pid=2144,fd=6))                          
LISTEN  0       200           127.0.0.1:5433           0.0.0.0:*      users:(("postgres",pid=2145,fd=6))                          
LISTEN  0       80            127.0.0.1:5432           0.0.0.0:*      users:(("postgres",pid=2147,fd=6))                          
LISTEN  0       4096          127.0.0.1:8384           0.0.0.0:*      users:(("syncthing",pid=6991,fd=26))                        
LISTEN  0       4096      127.0.0.53%lo:53             0.0.0.0:*      users:(("systemd-resolve",pid=1238,fd=15))                  
LISTEN  0       4096               [::]:8080              [::]:*      users:(("docker-proxy",pid=4375,fd=4))                      
LISTEN  0       5                  [::]:4330              [::]:*      users:(("pmlogger",pid=3230,fd=8))                          
LISTEN  0       10                    *:3389                 *:*      users:(("gnome-remote-de",pid=6531,fd=16))                  
LISTEN  0       4096                  *:22                   *:*      users:(("systemd",pid=1,fd=361))                            
LISTEN  0       4096               [::]:111               [::]:*      users:(("rpcbind",pid=1224,fd=6),("systemd",pid=1,fd=193))  
LISTEN  0       4096              [::1]:631               [::]:*      users:(("cupsd",pid=2038,fd=7))                             
LISTEN  0       5                  [::]:44321             [::]:*      users:(("pmcd",pid=2534,fd=3))                              
LISTEN  0       128                [::]:44323             [::]:*      users:(("pmproxy",pid=2559,fd=16))                          
LISTEN  0       128                [::]:44322             [::]:*      users:(("pmproxy",pid=2559,fd=14))                          
LISTEN  0       4096                  *:22000                *:*      users:(("syncthing",pid=6991,fd=16))                        
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6000:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14
docker: Error response from daemon: Conflict. The container name "/pg-server" is already in use by container "a616a13cc29b071849e0f317ba339e16349b607f65b864970270920c2c38398f". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6000:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14^C
root@tpt480:/home/serge# docker remove a616a13cc29b071849e0f317ba339e16349b607f65b864970270920c2c38398f
a616a13cc29b071849e0f317ba339e16349b607f65b864970270920c2c38398f
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6000:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14
23b7d6955da1d9988154a54e47f4e4ae3f9fc4a64d7b1e8952ba67abc770cda4
```

успешный запуск, проверка

```
root@tpt480:/home/serge# docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                                       NAMES
23b7d6955da1   postgres:14              "docker-entrypoint.s…"   7 seconds ago   Up 7 seconds   0.0.0.0:6000->5432/tcp, :::6000->5432/tcp   pg-server
1f2a9a3e38de   victorfadeev/spoof-dpi   "/root/.spoof-dpi/bi…"   2 years ago     Up 2 hours     0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   spoof-dpi
root@tpt480:/home/serge# docker run --rm --name pg-client --network pg-net -e PGPASSWORD=otus_pass postgres:14 psql -h pg-server -U postgres -c 'select version();' -c '\conninfo'
                                                       version                                                        
----------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.24 (Debian 14.24-1.pgdg13+2) on x86_64-pc-linux-gnu, compiled by gcc (Debian 14.2.0-19) 14.2.0, 64-bit
(1 row)

You are connected to database "postgres" as user "postgres" on host "pg-server" (address "172.18.0.2") at port "5432".
root@tpt480:/home/serge# docker run --rm -it  --name pg-client --network pg-net -e PGPASSWORD=otus_pass postgres:14 psql -h pg-server -U postgres -c 'select version();' -c '\conninfo'
                                                       version                                                        
----------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.24 (Debian 14.24-1.pgdg13+2) on x86_64-pc-linux-gnu, compiled by gcc (Debian 14.2.0-19) 14.2.0, 64-bit
(1 row)

You are connected to database "postgres" as user "postgres" on host "pg-server" (address "172.18.0.2") at port "5432".
```
используем штатный psql из контейнера, коннект есть ПГ живой.

Подключаемся к нему, вставляем исходные данные
```
root@tpt480:/home/serge# docker run --rm -it  --name pg-client --network pg-net -e PGPASSWORD=otus_pass postgres:14 psql -h pg-server -U postgres 
psql (14.24 (Debian 14.24-1.pgdg13+2))
Type "help" for help.

postgres=# \c
You are now connected to database "postgres" as user "postgres".
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg-server" (address "172.18.0.2") at port "5432".
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
postgres=# \q
```
останавливаем контейнер, удаляем. Проверяем нет ли хвостов от него.
```
root@tpt480:/home/serge# docker stop 23b7d6955da1
23b7d6955da1
root@tpt480:/home/serge# docker remove 23b7d6955da1
23b7d6955da1
root@tpt480:/home/serge# docker ps -a | grep 23b7d6955da1
```

Данные  в отдельной директории на месте.
```
root@tpt480:/home/serge# du -sm /opt/postgres/
42	/opt/postgres/

запускаем ПГ с "прежним" пробросом
root@tpt480:/home/serge# docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=otus_pass  -p 6000:5432   -v /opt/postgres:/var/lib/postgresql/data postgres:14
791159f3c26d7ef6538839eebcf0486b3c5f56be59bfaddd8ecbf89fa764edb2
```
подключаемся к нему клиентом в интерактивном режиме.
```
oot@tpt480:/home/serge# docker run --rm -it  --name pg-client --network pg-net -e PGPASSWORD=otus_pass postgres:14 psql -h pg-server -U postgres 
psql (14.24 (Debian 14.24-1.pgdg13+2))
Type "help" for help.

postgres=# \c
You are now connected to database "postgres" as user "postgres".
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg-server" (address "172.18.0.2") at port "5432".
```
делаем выборку из таблицы. Данные на месте
```
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

postgres=# 
```
выходим и подключаемся нативным клиентом с ноутбука
```
root@tpt480:/home/serge# psql -h localhost -U postgres  -p 6000
Password for user postgres: 
psql (16.9 (Ubuntu 16.9-0ubuntu0.24.04.1), server 14.24 (Debian 14.24-1.pgdg13+2))
Type "help" for help.

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

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "localhost" (address "127.0.0.1") at port "6000".
```
Данные на месте
