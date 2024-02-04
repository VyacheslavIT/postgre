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
  

Тест 1: в синхронном режиме с настройка ро умолчанию pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

tps = 3500.993491

```sql
 synchronous_commit
--------------------
 off
(1 row)
```
Тест 1 режим асинхронный настройки с pgtune pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

tps = 2285.493382

tps = 2287.461454

tps = 2262.970556

tps = 2285.540911

Тест 2 настройки по умолчанию режим асинхронный pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

tps = 2257.098643

Тест 2 увеличил значение shared_buffers до 1GB режим асинхронный pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

tps = 2284.790622

tps = 2260.621170

tps = 2277.590751

Тест 3 увеличил значение wal_buffers до 64MB режим асинхронный pgbench -c 50 -j 2 -P 10 -T 60 -U postgres postgres

tps = 2287.322406

tps = 2290.068770

tps = 2294.507911



-----------------------------------------------------
* написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему


----------------------------------------------------
* Задание со *:
* аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)




1:  установка curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench

 CREATE USER sbtest WITH PASSWORD 'password';
 
 CREATE DATABASE sbtest;
 
GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbtest;


2: подготовим 100 таблиц по 10000 строк : sysbench --db-driver=pgsql  --tables=10   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua prepare


3: Запустим тест sysbench --db-driver=pgsql  --tables=10   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua run


```sql

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
