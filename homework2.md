* создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/e944a1a2-0492-4e03-b2c4-e13cdc61c439)
---------------------
* поставить на нем Docker Engine
*curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker*
  
>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/7d39f4b1-e138-4455-bdb2-bd4f11d85f4c)
--------------------------
* развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
 *sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15* 
>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/ede9bcef-9263-41a3-ab99-eccddb47d292)

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/f56504bf-9a35-4481-bffe-97a2c458b98e)


>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/ad16690d-772f-40f1-983a-608bc0bb036f)
------------------------------------
* подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
  *sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres*
  
```sql
CREATE TABLE test (i serial, amount int);
INSERT INTO test(amount) VALUES (100);
INSERT INTO test(amount) VALUES (500);
```

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/3e471dc9-4fb5-4167-bf4f-a0dd9a64f3fc)

------------------------------------
* подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/5b788b3f-d6b4-43e0-b2dc-5ec3b3d503ed)

* удалить контейнер с сервером
  
sudo docker stop 77b977a23444

sudo docker rm 77b977a23444
  
>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/6ddb39bf-f8ee-4dd2-a741-e3e880a2b0cd)

-------------------------------------

* создать его заново

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/182aee12-a0ef-49df-9a03-82c62b24bfb5)
 ---------------------------
 * подключится снова из контейнера с клиентом к контейнеру с сервером

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/e1c13f91-bafd-42be-93a9-9b7f5c865aa5)
* проверить, что данные остались на месте

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/70e554c7-6fc8-4650-91c0-f1708040f054)

>![image](https://github.com/VyacheslavIT/postgre/assets/136000255/96296ca7-691f-4f55-ac4a-5c03d84c567c)



  
