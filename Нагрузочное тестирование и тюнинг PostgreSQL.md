* развернуть виртуальную машину любым удобным способом
  
Параметры системы ОЗУ 4GB, CPU 2, SSD


  
------------------------------------------------------
* поставить на неё PostgreSQL 15 любым способом

![image](https://github.com/VyacheslavIT/postgre/assets/136000255/9f038f6b-264b-419a-9ab2-257ec9b0d119)

-----------------------------------------------------
* настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины

Выставил настройки из pgtune

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
  
tps = 4325.773212   Настройки по умолчанию и настройки с pgtune дают плюс минус одинаковую производительность, особой разницы в производительности нет, думаю эффект будет заметен на больших базах и на железе с более высокой конфигурацией.

Брал настройки с pgtune, думаю это основные настройки которые нужно выставить.

Увеличение значений shared_buffers и effective_cache_size может улучшить производительность, поскольку это позволяет большему количеству данных храниться в памяти, что уменьшает количество чтения и записи на диск.

Увеличение maintenance_work_mem также может улучшить производительность при выполнении операций обслуживания базы данных, таких как ребилд индексов.

Установка checkpoint_completion_target на значение 0.9 позволяет уменьшить количество времени, затрачиваемого на исполнение процесса контрольной точки, что может улучшить производительность.

Установка default_statistics_target на 100 помогает PostgreSQL лучше собирать статистику о данных и использовать ее для оптимизации запросов.

Установка random_page_cost на 1.1 и effective_io_concurrency на 200 позволяет учитывать случайный доступ к данным и обрабатывать операции ввода-вывода эффективнее.

Установка work_mem на 2621kB управляет объемом памяти, выделенной на выполнение каждого оператора сортировки или хэширования.

Установка параметров huge_pages и wal_size может также влиять на производительность, но они зависят от специфики вашей системы и требуют дополнительной настройки.

Думаю все равно нужно тестировать настройки под каждую БД и корректировать, также многое зависит от железа.
На виртуальной машине Яндекс тест выдавал в асинхронном режиме в среднем tps = 2300 на моей виртуальной tps = 4325.773212


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

Данная утилита показала, что настройки по умолчанию дают немного большую производительность чем настройки с pgtune.


1:  установка curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench
```sql

 CREATE USER sbtest WITH PASSWORD 'password';
 
 CREATE DATABASE sbtest;
 
GRANT ALL PRIVILEGES ON DATABASE sbtest TO sbtest;
```

2: подготовим 100 таблиц по 10000 строк : sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 /usr/share/sysbench/oltp_read_write.lua prepare


3: Запустим тест sysbench --db-driver=pgsql  --tables=100   --pgsql-host=localhost --pgsql-port=5433 --pgsql-user=sbtest --pgsql-password=password --pgsql-db=sbtest --report-interval=2 -threads=2 /usr/share/sysbench/oltp_read_write.lua run


```sql
Тест 1: в синхронном режиме с настройка по умолчанию


Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 695.59 qps: 13926.38 (r/w/o: 9751.82/2782.38/1392.19) lat (ms,95%): 5.37 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1075.09 qps: 21494.79 (r/w/o: 15044.26/4300.36/2150.18) lat (ms,95%): 2.61 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 984.99 qps: 19699.30 (r/w/o: 13789.86/3939.96/1969.48) lat (ms,95%): 3.07 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 1074.50 qps: 21497.41 (r/w/o: 15049.43/4298.48/2149.49) lat (ms,95%): 2.91 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1064.81 qps: 21296.17 (r/w/o: 14907.32/4259.23/2129.62) lat (ms,95%): 2.86 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            137102
        write:                           39172
        other:                           19586
        total:                           195860
    transactions:                        9793   (978.34 per sec.)
    queries:                             195860 (19566.84 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0088s
    total number of events:              9793

Latency (ms):
         min:                                    0.92
         avg:                                    2.04
         max:                                   47.85
         95th percentile:                        3.07
         sum:                                19983.32

Threads fairness:
    events (avg/stddev):           4896.5000/502.50
    execution time (avg/stddev):   9.9917/0.00



```

```sql
Тест 2: в синхронном режиме настройки с  pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 829.01 qps: 16591.77 (r/w/o: 11616.69/3316.06/1659.03) lat (ms,95%): 5.57 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 871.57 qps: 17436.36 (r/w/o: 12204.95/3488.27/1743.14) lat (ms,95%): 3.25 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1006.00 qps: 20117.44 (r/w/o: 14081.46/4023.99/2011.99) lat (ms,95%): 3.13 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 964.50 qps: 19288.01 (r/w/o: 13503.01/3856.00/1929.00) lat (ms,95%): 3.19 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1011.46 qps: 20232.17 (r/w/o: 14162.92/4046.33/2022.92) lat (ms,95%): 2.97 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            131152
        write:                           37472
        other:                           18736
        total:                           187360
    transactions:                        9368   (936.45 per sec.)
    queries:                             187360 (18729.00 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0028s
    total number of events:              9368

Latency (ms):
         min:                                    0.95
         avg:                                    2.13
         max:                                   13.04
         95th percentile:                        3.36
         sum:                                19986.16

Threads fairness:
    events (avg/stddev):           4684.0000/259.00
    execution time (avg/stddev):   9.9931/0.00



```




```sql
Тест 3 режим асинхронный настройки с pgtune

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 1063.88 qps: 21281.06 (r/w/o: 14896.79/4255.51/2128.76) lat (ms,95%): 3.43 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1372.57 qps: 27460.81 (r/w/o: 19225.42/5490.26/2745.13) lat (ms,95%): 2.14 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1269.03 qps: 25381.00 (r/w/o: 17766.35/5076.60/2538.05) lat (ms,95%): 2.35 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 1312.99 qps: 26257.28 (r/w/o: 18378.35/5252.96/2625.98) lat (ms,95%): 2.61 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1207.49 qps: 24145.32 (r/w/o: 16902.38/4828.46/2414.48) lat (ms,95%): 2.30 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            174356
        write:                           49816
        other:                           24908
        total:                           249080
    transactions:                        12454  (1244.89 per sec.)
    queries:                             249080 (24897.77 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0033s
    total number of events:              12454

Latency (ms):
         min:                                    0.65
         avg:                                    1.60
         max:                                   15.00
         95th percentile:                        2.43
         sum:                                19983.78

Threads fairness:
    events (avg/stddev):           6227.0000/210.00
    execution time (avg/stddev):   9.9919/0.00


Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 590.87 qps: 11825.80 (r/w/o: 8279.11/2364.46/1182.23) lat (ms,95%): 9.39 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1376.75 qps: 27535.07 (r/w/o: 19274.55/5506.51/2754.01) lat (ms,95%): 2.57 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 2205.48 qps: 44117.10 (r/w/o: 30882.72/8823.42/4410.96) lat (ms,95%): 1.39 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 2210.04 qps: 44200.82 (r/w/o: 30941.07/8839.66/4420.08) lat (ms,95%): 1.32 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 2209.44 qps: 44185.84 (r/w/o: 30930.69/8835.77/4419.38) lat (ms,95%): 1.32 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            240618
        write:                           68747
        other:                           34375
        total:                           343740
    transactions:                        17187  (1717.94 per sec.)
    queries:                             343740 (34358.88 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0035s
    total number of events:              17187

Latency (ms):
         min:                                    0.66
         avg:                                    1.16
         max:                                   23.19
         95th percentile:                        2.52
         sum:                                19977.33

Threads fairness:
    events (avg/stddev):           8593.5000/340.50
    execution time (avg/stddev):   9.9887/0.00




```
```sql
Тест 4 режим асинхронный настройки  по умолчанию

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 1124.88 qps: 22506.53 (r/w/o: 15756.27/4499.51/2250.75) lat (ms,95%): 3.30 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 1865.60 qps: 37311.56 (r/w/o: 26117.44/7463.41/3730.71) lat (ms,95%): 2.30 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1408.99 qps: 28180.85 (r/w/o: 19727.40/5634.97/2818.49) lat (ms,95%): 2.39 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 2365.01 qps: 47305.77 (r/w/o: 33113.69/9461.55/4730.53) lat (ms,95%): 1.39 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 2390.98 qps: 47813.53 (r/w/o: 33469.67/9561.91/4781.95) lat (ms,95%): 1.25 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            256396
        write:                           73255
        other:                           36629
        total:                           366280
    transactions:                        18314  (1830.75 per sec.)
    queries:                             366280 (36615.09 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0026s
    total number of events:              18314

Latency (ms):
         min:                                    0.63
         avg:                                    1.09
         max:                                   12.19
         95th percentile:                        2.35
         sum:                                19979.54

Threads fairness:
    events (avg/stddev):           9157.0000/401.00
    execution time (avg/stddev):   9.9898/0.00


```

```sql
Тест 5: в синхронном режиме настройки с  pgtune

wal_buffers = 64MB
effective_io_concurrency = 1

Initializing worker threads...

Threads started!

[ 2s ] thds: 2 tps: 840.85 qps: 16825.07 (r/w/o: 11778.95/3363.42/1682.71) lat (ms,95%): 4.57 err/s: 0.00 reconn/s: 0.00
[ 4s ] thds: 2 tps: 972.74 qps: 19452.39 (r/w/o: 13615.92/3890.98/1945.49) lat (ms,95%): 2.76 err/s: 0.00 reconn/s: 0.00
[ 6s ] thds: 2 tps: 1109.49 qps: 22190.88 (r/w/o: 15533.91/4437.98/2218.99) lat (ms,95%): 3.30 err/s: 0.00 reconn/s: 0.00
[ 8s ] thds: 2 tps: 1100.95 qps: 22025.95 (r/w/o: 15420.27/4403.79/2201.90) lat (ms,95%): 2.66 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 2 tps: 1041.05 qps: 20818.01 (r/w/o: 14571.71/4164.20/2082.10) lat (ms,95%): 3.02 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            141862
        write:                           40532
        other:                           20266
        total:                           202660
    transactions:                        10133  (1012.79 per sec.)
    queries:                             202660 (20255.77 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.0041s
    total number of events:              10133

Latency (ms):
         min:                                    0.90
         avg:                                    1.97
         max:                                   19.37
         95th percentile:                        3.30
         sum:                                19986.68

Threads fairness:
    events (avg/stddev):           5066.5000/61.50
    execution time (avg/stddev):   9.9933/0.00



```
