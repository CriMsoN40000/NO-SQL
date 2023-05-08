# CLICKHOUSE #

## Задание: ##
- **Развернуть БД** 

Развертывание БД выполняем с помощью ВМ Yandex Cloud
````
apt-get install -y apt-transport-https ca-certificates dirmngr
apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
apt-get update

apt-get install -y clickhouse-server clickhouse-client

service clickhouse-server start
````

Проверяем установку

````
root@clickhouse:~# service clickhouse-server status
● clickhouse-server.service - ClickHouse Server (analytic DBMS for big data)
     Loaded: loaded (/lib/systemd/system/clickhouse-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-05-08 13:24:27 UTC; 8min ago
   Main PID: 934 (clickhouse-serv)
      Tasks: 274 (limit: 3361)
     Memory: 521.8M
        CPU: 5.073s
     CGroup: /system.slice/clickhouse-server.service
             ├─720 clickhouse-watchdog "" "" "" "" "" "" "" --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-serv>
             └─934 /usr/bin/clickhouse-server --config=/etc/clickhouse-server/config.xml --pid-file=/run/clickhouse-server/clickhouse-server.pid

May 08 13:24:11 clickhouse clickhouse-server[720]: Logging errors to /var/log/clickhouse-server/clickhouse-server.err.log
May 08 13:24:11 clickhouse systemd[1]: clickhouse-server.service: Supervising process 934 which is not our child. We'll most likely not notice when it e>
May 08 13:24:25 clickhouse clickhouse-server[934]: Processing configuration file '/etc/clickhouse-server/config.xml'.
May 08 13:24:25 clickhouse clickhouse-server[934]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/config.xml'.
May 08 13:24:25 clickhouse clickhouse-server[934]: Processing configuration file '/etc/clickhouse-server/users.xml'.
May 08 13:24:25 clickhouse clickhouse-server[934]: Merging configuration file '/etc/clickhouse-server/users.d/default-password.xml'.
May 08 13:24:25 clickhouse clickhouse-server[934]: Saved preprocessed configuration to '/var/lib/clickhouse/preprocessed_configs/users.xml'.
May 08 13:24:27 clickhouse systemd[1]: Started ClickHouse Server (analytic DBMS for big data).

````

Проверим, что все развернулось и работает

`````
clickhouse.ru-central1.internal :) show databases

SHOW DATABASES

Query id: ee3fa632-3617-482d-95d5-3a89a5d81545

┌─name───────────────┐
│ INFORMATION_SCHEMA │
│ default            │
│ information_schema │
│ system             │
└────────────────────┘

lickhouse.ru-central1.internal :) show tables limit 5

SHOW TABLES LIMIT 5

Query id: 33a01121-fc5c-4a32-8a06-9a4217fede4b

┌─name───────────────────────────┐
│ aggregate_function_combinators │
│ asynchronous_inserts           │
│ asynchronous_metric_log        │
│ asynchronous_metrics           │
│ backups                        │
└────────────────────────────────┘

5 rows in set. Elapsed: 0.002 sec. 
`````

- **Выполнить импорт тестовой БД**

Создадим директорию для загрузки дампов в БД

````
alext@clickhouse:~$ mkdir testdata
alext@clickhouse:~$ cd testdata/
````

Скачаем дампы 

````
curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv
````



выполнить несколько запросов и оценить скорость выполнения.
развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов
