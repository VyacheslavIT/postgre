* развернуть виртуальную машину любым удобным способом



  
------------------------------------------------------
* поставить на неё PostgreSQL 15 любым способом

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/9f038f6b-264b-419a-9ab2-257ec9b0d119)

-----------------------------------------------------
* настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины

Выставил настройки из pg_tune

```sql

max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB

```

и асинхронный режим

----------------------------------------------------
* нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)

  подготовка pgbench -i -p 5433 -s 10
  

Тест 1: в синхронном режиме с настройка по умолчанию pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 3500.993491

Тест 2: в синхронном режиме настройки с  pgtune

tps = 3304.568552

```sql
postgres=# show synchronous_commit;
 synchronous_commit 
--------------------
 off
(1 row)

```
Тест 3 режим асинхронный настройки  по умолчанию  pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 4302.523401

Тест 4 режим асинхронный настройки с  pgtune  pgbench -c 50 -j 2 -P 10 -p 5433 -T 60 -U postgres postgres

tps = 4325.773212 


-----------------------------------------------------
* написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
  
tps = 4325.773212 
```sql
max_connections = 100
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 256MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 2621kB
huge_pages = off
min_wal_size = 1GB
max_wal_size = 4GB
```

----------------------------------------------------
* Задание со *:
* аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)




1:  установка curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```sql

 CREATE USER sbtest WITH PASSWORD 'password';
 
 CREATE DATABASE sbtest;
 
GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbtest;
```

2: подготовим 100 таблиц по 10000 строк : sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua prepare


3: Запустим тест sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua run


```sql
Тест 1: в синхронном режиме с настройка по умолчанию

Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 701.11 qps: 14031.61 (r/w/o: 9822.48/2806.42/1402.71) lat (ms,95%): 2.48 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 863.52 qps: 17270.45 (r/w/o: 12089.32/3454.09/1727.05) lat (ms,95%): 1.73 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 830.18 qps: 16594.68 (r/w/o: 11615.58/3318.74/1660.37) lat (ms,95%): 1.89 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 866.58 qps: 17340.51 (r/w/o: 12139.06/3468.30/1733.15) lat (ms,95%): 1.86 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 873.81 qps: 17476.12 (r/w/o: 12233.28/3495.22/1747.61) lat (ms,95%): 1.89 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            115808
        write:                           33088
        other:                           16544
        total:                           165440
    transactions:                        8272   (826.91 per sec.)
    queries:                             165440 (16538.28 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0023s
    total number of events:              8272

Latency (ms):
         min:                                    0.86
         avg:                                    1.21
         max:                                   15.96
         95th percentile:                        1.96
         sum:                                 9988.98

Threads fairness:
    events (avg/stddev):           8272.0000/0.00
    execution time (avg/stddev):   9.9890/0.00



```

```sql
Тест 2: в синхронном режиме настройки с  pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 619.13 qps: 12383.06 (r/w/o: 8667.79/2476.51/1238.76) lat (ms,95%): 2.57 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 770.52 qps: 15415.86 (r/w/o: 10792.75/3082.07/1541.04) lat (ms,95%): 2.03 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 781.45 qps: 15628.90 (r/w/o: 10940.23/3125.78/1562.89) lat (ms,95%): 1.96 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 770.61 qps: 15415.76 (r/w/o: 10790.08/3084.45/1541.23) lat (ms,95%): 1.79 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 778.46 qps: 15569.12 (r/w/o: 10898.38/3113.82/1556.91) lat (ms,95%): 1.73 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            104188
        write:                           29768
        other:                           14884
        total:                           148840
    transactions:                        7442   (744.03 per sec.)
    queries:                             148840 (14880.58 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0014s
    total number of events:              7442

Latency (ms):
         min:                                    0.97
         avg:                                    1.34
         max:                                   10.79
         95th percentile:                        2.03
         sum:                                 9988.86

Threads fairness:
    events (avg/stddev):           7442.0000/0.00
    execution time (avg/stddev):   9.9889/0.00



```




```sql
Тест 3 режим асинхронный настройки с pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 1127.86 qps: 22564.74 (r/w/o: 15797.07/4511.45/2256.22) lat (ms,95%): 1.30 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 1200.07 qps: 24000.37 (r/w/o: 16799.96/4800.27/2400.14) lat (ms,95%): 1.27 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 1255.50 qps: 25103.47 (r/w/o: 17570.98/5021.99/2510.50) lat (ms,95%): 1.21 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 1269.27 qps: 25385.49 (r/w/o: 17769.84/5077.10/2538.55) lat (ms,95%): 1.06 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 1237.22 qps: 24753.88 (r/w/o: 17328.07/4950.88/2474.94) lat (ms,95%): 1.32 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            170548
        write:                           48728
        other:                           24364
        total:                           243640
    transactions:                        12182  (1217.82 per sec.)
    queries:                             243640 (24356.42 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0022s
    total number of events:              12182

Latency (ms):
         min:                                    0.63
         avg:                                    0.82
         max:                                   22.55
         95th percentile:                        1.23
         sum:                                 9986.69

Threads fairness:
    events (avg/stddev):           12182.0000/0.00
    execution time (avg/stddev):   9.9867/0.00


```
```sql
Тест 3 режим асинхронный настройки  по умолчанию
Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 1164.14 qps: 23292.38 (r/w/o: 16305.01/4658.58/2328.79) lat (ms,95%): 1.08 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 1252.24 qps: 25035.26 (r/w/o: 17524.33/5006.95/2503.98) lat (ms,95%): 0.90 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 1288.59 qps: 25780.23 (r/w/o: 18047.21/5155.35/2577.67) lat (ms,95%): 0.87 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 1191.47 qps: 23820.93 (r/w/o: 16673.60/4764.89/2382.44) lat (ms,95%): 1.23 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 1278.97 qps: 25582.30 (r/w/o: 17908.01/5115.86/2558.43) lat (ms,95%): 0.89 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            172942
        write:                           49412
        other:                           24706
        total:                           247060
    transactions:                        12353  (1234.95 per sec.)
    queries:                             247060 (24699.05 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0020s
    total number of events:              12353

Latency (ms):
         min:                                    0.65
         avg:                                    0.81
         max:                                    4.54
         95th percentile:                        1.01
         sum:                                 9987.39

Threads fairness:
    events (avg/stddev):           12353.0000/0.00
    execution time (avg/stddev):   9.9874/0.00


```
