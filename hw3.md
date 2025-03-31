```
postgres=# SELECT relname, n_live_tup, n_dead_tup,

trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%",
last_autovacuum

FROM pg_stat_user_tables WHERE relname = 'test';

relname | n_live_tup | n_dead_tup | ratio% | last_autovacuum

---------+------------+------------+--------+-------------------------------

test | 1581068 | 9995694 | 632 | 2025-03-19
11:47:59.324632+03

(1 row)




Time: 1.057 ms

postgres=# vacuum;

VACUUM

Time: 2392.541 ms (00:02.393)

postgres=# SELECT relname, n_live_tup, n_dead_tup,

trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%",
last_autovacuum

FROM pg_stat_user_tables WHERE relname = 'test';

relname | n_live_tup | n_dead_tup | ratio% | last_autovacuum

---------+------------+------------+--------+-------------------------------

test | 1000000 | 0 | 0 | 2025-03-19
11:47:59.324632+03

(1 row)

Time: 0.880 ms
```

 
 
Объяснение полученного результата:
После отключения автовакуума количество мертвых строк (n_dead_tup) в таблице будет увеличиваться ибо автовакуум больше не будет автоматически очищать их. Это приведет к увеличению размера таблицы, БД будет хранить старые версии строк до тех пор, пока не будет выполнена ручная очистка(vacuum).