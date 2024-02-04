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


2: подготовим 10 таблиц по 10000 строк : sysbench --db-driver=pgsql  --tables=10   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua prepare


3: Запустим тест sysbench --db-driver=pgsql  --tables=10   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua run


```sql
Initializing worker threads...

Threads started!

[ 2s ] thds: 1 tps: 950.25 qps: 19012.04 (r/w/o: 13310.03/3801.01/1901.00) lat (ms,95%): 1.55 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 1 tps: 945.23 qps: 18899.16 (r/w/o: 13227.76/3780.93/1890.47) lat (ms,95%): 1.50 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 1 tps: 831.48 qps: 16635.19 (r/w/o: 11646.28/3325.94/1662.97) lat (ms,95%): 1.89 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 1 tps: 915.01 qps: 18298.66 (r/w/o: 12808.61/3660.03/1830.02) lat (ms,95%): 1.61 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 1 tps: 854.50 qps: 17090.05 (r/w/o: 11963.03/3418.01/1709.00) lat (ms,95%): 1.96 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            125930
        write:                           35980
        other:                           17990
        total:                           179900
    transactions:                        8995   (899.30 per sec.)
    queries:                             179900 (17986.01 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0013s
    total number of events:              8995

Latency (ms):
         min:                                    0.81
         avg:                                    1.11
         max:                                   38.36
         95th percentile:                        1.76
         sum:                                 9988.38

Threads fairness:
    events (avg/stddev):           8995.0000/0.00
    execution time (avg/stddev):   9.9884/0.00



```
