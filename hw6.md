## Подготовка: 

```
postgres@pg-db:~$ psql -c "CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'secret$123';"
CREATE ROLE

postgres@pg-db:~$ psql -c "SELECT pg_create_physical_replication_slot('test');"
 pg_create_physical_replication_slot
-------------------------------------
 (test,)
(1 row)

postgres@pg-db:~$ pg_createcluster 15 lab6

sheludchenko.a4@postgres:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  lab6    5433 online postgres /var/lib/postgresql/15/lab6 /var/log/postgresql/postgresql-15-lab6.log
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres@pg-db:~$ pg_ctlcluster 15 lab6 stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-lab6

postgres@pg-db:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  lab6    5433 down   postgres /var/lib/postgresql/15/lab6 /var/log/postgresql/postgresql-15-lab6.log
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres@pg-db:~$ rm -rf /var/lib/postgresql/15/lab6

postgres@pg-db:~$ pg_lsclusters
Ver Cluster Port Status Owner     Data directory              Log file
15  lab6    5433 down   <unknown> /var/lib/postgresql/15/lab6 /var/log/postgresql/postgresql-15-lab6.log
15  main    5432 online postgres  /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres@pg-db:~$ pg_basebackup -h localhost -p 5432 -U replicator -R -S test -D /var/lib/postgresql/15/lab6

postgres@pg-db:~$ pg_lsclusters
Ver Cluster Port Status        Owner    Data directory              Log file
15  lab6    5433 down,recovery postgres /var/lib/postgresql/15/lab6 /var/log/postgresql/postgresql-15-lab6.log
15  main    5432 online        postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```


## Тесты

Исходный тест на запись, без реплики:

```
postgres@pg-db:~$ cat > ~/workload2.sql << EOL
INSERT INTO book.tickets (fkRide, fio, contact, fkSeat)
VALUES (
        ceil(random()*100)
        , (array(SELECT fam FROM book.fam))[ceil(random()*110)]::text || ' ' ||
    (array(SELECT nam FROM book.nam))[ceil(random()*110)]::text
    ,('{"phone":"+7' || (1000000000::bigint + floor(random()*9000000000)::bigint)::text || '"}')::jsonb
    , ceil(random()*100));

EOL

postgres@pg-db:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 50261
number of failed transactions: 0 (0.000%)
latency average = 1.592 ms
initial connection time = 11.684 ms
tps = 5026.257322 (without initial connection time)
```


На чтение:

```
postgres@pg-db:~$ cat > ~/workload.sql << EOL
\set r random(1, 5000000)
SELECT id, fkRide, fio, contact, fkSeat FROM book.tickets WHERE id = :r;
EOL

postgres@pg-db:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 266231
number of failed transactions: 0 (0.000%)
latency average = 0.300 ms
initial connection time = 14.773 ms
tps = 26661.460509 (without initial connection time)
```

## Запуск реплики

```
postgres@pg-db:~$ pg_ctlcluster 15 lab6 start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-lab6

postgres@pg-db:~$ pg_lsclusters
Ver Cluster Port Status          Owner    Data directory              Log file
15  lab6    5433 online,recovery postgres /var/lib/postgresql/15/lab6 /var/log/postgresql/postgresql-15-lab6.log
15  main    5432 online          postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log

postgres@pg-db:~$ psql -p 5433 -d thai -c "select pg_is_in_recovery();"
 pg_is_in_recovery
-------------------
 t
(1 row)
```


### Тесты
Тест записи с репликой:
```
postgres@pg-db:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload2.sql -n -U postgres -p 5432 thai
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload2.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 31694
number of failed transactions: 0 (0.000%)
latency average = 2.523 ms
initial connection time = 11.740 ms
tps = 3170.418338 (without initial connection time)
```


Тест чтения с репликой:
```
postgres@pg-db:~$ pgbench -c 8 -j 4 -T 10 -f ~/workload.sql -n -U postgres thai
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 253001
number of failed transactions: 0 (0.000%)
latency average = 0.316 ms
initial connection time = 10.881 ms
tps = 25304.809225 (without initial connection time)
```


### Итоги
```
 Тест на вставку значений 5026 -> 3170
 Тест на скорость чтения 26661 -> 25304
```
После добавления асинхронной реплики, видим падение в скорости на вставку на 58%. Скорость на чтение на лидере и реплике в пределах погрешности