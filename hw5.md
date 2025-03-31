##  default postgres
```
postgres@pg-db:~$ pgbench -c 8 -j 4 -T 10 -f /home/bogatyrev.andrey8/workload.sql -n -U postgres thai
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /home/bogatyrev.andrey8/workload.sql
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 275537
number of failed transactions: 0 (0.000%)
latency average = 0.290 ms
initial connection time = 10.895 ms
tps = 27580.668378 (without initial connection time)
```

##  pgbouncer default mode: session
```
postgres@pg-db:~$ pgbench -c 120 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h 127.0.0.1 thai
Password:
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 120
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 98001
number of failed transactions: 0 (0.000%)
latency average = 10.865 ms
initial connection time = 1151.123 ms
tps = 11044.478026 (without initial connection time)
```

##  pgbouncer: transaction
```
postgres@pg-db:~$ pgbench -c 120 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h 127.0.0.1 thai
Password:
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 120
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 107611
number of failed transactions: 0 (0.000%)
latency average = 9.965 ms
initial connection time = 1083.215 ms
tps = 12041.761843 (without initial connection time)
```

##  pgbouncer: statement
```
postgres@pg-db:~$ pgbench -c 120 -j 4 -T 10 -f ~/workload.sql -n -U postgres -p 6432 -h 127.0.0.1 thai
Password:
pgbench (15.12 (Debian 15.12-0+deb12u2))
transaction type: /var/lib/postgresql/workload.sql
scaling factor: 1
query mode: simple
number of clients: 120
number of threads: 4
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 111397
number of failed transactions: 0 (0.000%)
latency average = 9.665 ms
initial connection time = 1042.873 ms
tps = 12415.536691 (without initial connection time)
```


## check config:
```
> show config;
 max_client_conn           | 1000                                                   | 100                                                    | yes
 max_db_connections        | 1000                                                   | 0                                                      | yes
 max_packet_size           | 2147483647                                             | 2147483647                                             | yes
 max_user_connections      | 1000                                                   | 0                                                      | yes
 min_pool_size             | 0                                                      | 0                                                      | yes
 pidfile                   | /var/run/postgresql/pgbouncer.pid                      |                                                        | no
 pkt_buf                   | 4096                                                   | 4096                                                   | no
 pool_mode                 | statement                                              | session                                                | yes
```




