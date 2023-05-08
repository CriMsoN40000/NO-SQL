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

Создаем БД для загрузки тестовых данных и таблицу

```
clickhouse.ru-central1.internal :) CREATE DATABASE hits

CREATE DATABASE hits

Query id: 8c965c9f-fab3-41e1-b2cd-5f6bb1b404ee

Ok.

0 rows in set. Elapsed: 0.023 sec. 

CREATE TABLE hits.hits_v1
(
    `WatchID` UInt64,
    `JavaEnable` UInt8,
    `Title` String,
    `GoodEvent` Int16,
    `EventTime` DateTime,
    `EventDate` Date,
    `CounterID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RegionID` UInt32,
    `UserID` UInt64,
    `CounterClass` Int8,
    `OS` UInt8,
    `UserAgent` UInt8,
    `URL` String,
    `Referer` String,
    `URLDomain` String,
    `RefererDomain` String,
    `Refresh` UInt8,
    `IsRobot` UInt8,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `FlashMinor2` String,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` FixedString(2),
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `MobilePhone` UInt8,
    `MobilePhoneModel` String,
    `Params` String,
    `IPNetworkID` UInt32,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `IsArtifical` UInt8,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `ClientTimeZone` Int16,
    `ClientEventTime` DateTime,
    `SilverlightVersion1` UInt8,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion3` UInt32,
    `SilverlightVersion4` UInt16,
    `PageCharset` String,
    `CodeVersion` UInt32,
    `IsLink` UInt8,
    `IsDownload` UInt8,
    `IsNotBounce` UInt8,
    `FUniqID` UInt64,
    `HID` UInt32,
    `IsOldCounter` UInt8,
    `IsEvent` UInt8,
    `IsParameter` UInt8,
    `DontCountHits` UInt8,
    `WithHash` UInt8,
    `HitColor` FixedString(1),
    `UTCEventTime` DateTime,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `WindowName` Int32,
    `OpenerName` Int32,
    `HistoryLength` Int16,
    `BrowserLanguage` FixedString(2),
    `BrowserCountry` FixedString(2),
    `SocialNetwork` String,
    `SocialAction` String,
    `HTTPError` UInt16,
    `SendTiming` Int32,
    `DNSTiming` Int32,
    `ConnectTiming` Int32,
    `ResponseStartTiming` Int32,
    `ResponseEndTiming` Int32,
    `FetchTiming` Int32,
    `RedirectTiming` Int32,
    `DOMInteractiveTiming` Int32,
    `DOMContentLoadedTiming` Int32,
    `DOMCompleteTiming` Int32,
    `LoadEventStartTiming` Int32,
    `LoadEventEndTiming` Int32,
    `NSToDOMContentLoadedTiming` Int32,
    `FirstPaintTiming` Int32,
    `RedirectCount` Int8,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `ParamPrice` Int64,
    `ParamOrderID` String,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `GoalsReached` Array(UInt32),
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `RefererHash` UInt64,
    `URLHash` UInt64,
    `CLID` UInt32,
    `YCLID` UInt64,
    `ShareService` String,
    `ShareURL` String,
    `ShareTitle` String,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `IslandID` FixedString(16),
    `RequestNum` UInt32,
    `RequestTry` UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)

clickhouse.ru-central1.internal :) show tables

SHOW TABLES

Query id: 3b258a9e-f5e3-4c73-944f-4ece62c08a75

┌─name────┐
│ hits_v1 │
└─────────┘

1 row in set. Elapsed: 0.001 sec. 
```

Загружаем данные

````
alext@clickhouse:~/testdata$ clickhouse-client --password 99Compat --query "INSERT INTO hits.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv

clickhouse.ru-central1.internal :) select count(1) from hits_v1;

SELECT count(1)
FROM hits_v1

Query id: 13e786c6-2bbd-47aa-9f44-c4e840dfc5fa

┌─count()─┐
│ 8873898 │
└─────────┘

1 row in set. Elapsed: 0.002 sec. 
````
- **Выполнить несколько запросов и оценить скорость выполнения**

````
clickhouse.ru-central1.internal :) select MobilePhoneModel, avg(Age)  from hits_v1 group by MobilePhoneModel limit 10;

SELECT
    MobilePhoneModel,
    avg(Age)
FROM hits_v1
GROUP BY MobilePhoneModel
LIMIT 10

Query id: 5dba05df-7c35-494c-87f1-828ffac97b30

┌─MobilePhoneModel─┬───────────avg(Age)─┐
│                  │ 23.813933046276528 │
│ life             │                  0 │
│ C6-01            │                  0 │
│ TV               │                  0 │
│ BlackBer         │  34.92307692307692 │
│ IQ431            │ 17.208333333333332 │
│ Samsung          │ 23.641011235955055 │
│ IQ43             │                  0 │
│ S6500s           │                 39 │
│ 6720             │ 1.7763157894736843 │
└──────────────────┴────────────────────┘

10 rows in set. Elapsed: 0.083 sec. Processed 8.87 million rows, 96.85 MB (106.55 million rows/s., 1.16 GB/s.)

clickhouse.ru-central1.internal :) 

clickhouse.ru-central1.internal :) select SearchPhrase, avg(Age) from hits_v1 where SearchPhrase is not null group by SearchPhrase limit 10;

SELECT
    SearchPhrase,
    avg(Age)
FROM hits_v1
WHERE SearchPhrase IS NOT NULL
GROUP BY SearchPhrase
LIMIT 10

Query id: 6e24b50e-d322-427e-a616-ec32dbdfce90

┌─SearchPhrase─┬───────────avg(Age)─┐
│              │ 23.270129149381106 │
│ britness     │                 16 │
│ перв         │ 26.857142857142858 │
│ ртр          │                 16 │
│ гада         │                 55 │
│ см           │                 16 │
│ ми-9         │ 15.333333333333334 │
│ колепный     │                 21 │
│ бабченки     │                 39 │
│ пласть 4     │               19.5 │
└──────────────┴────────────────────┘

10 rows in set. Elapsed: 0.130 sec. Processed 8.87 million rows, 121.57 MB (68.11 million rows/s., 933.08 MB/s.)

````

- **Развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов**

Подготавливаем базу данных и создаем таблицу для загрузки данных
````
clickhouse.ru-central1.internal :) use taxi_test

USE taxi_test

Query id: e7d0b17e-6a5d-494b-a5bd-a7bd3a61b813

Ok.

0 rows in set. Elapsed: 0.001 sec. 

CREATE TABLE trips
(
    `trip_id` UInt32,
    `vendor_id` Enum8('1' = 1, '2' = 2, '3' = 3, '4' = 4, 'CMT' = 5, 'VTS' = 6, 'DDS' = 7, 'B02512' = 10, 'B02598' = 11, 'B02617' = 12, 'B02682' = 13, 'B02764' = 14, '' = 15),
    `pickup_date` Date,
    `pickup_datetime` DateTime,
    `dropoff_date` Date,
    `dropoff_datetime` DateTime,
    `store_and_fwd_flag` UInt8,
    `rate_code_id` UInt8,
    `pickup_longitude` Float64,
    `pickup_latitude` Float64,
    `dropoff_longitude` Float64,
    `dropoff_latitude` Float64,
    `passenger_count` UInt8,
    `trip_distance` Float64,
    `fare_amount` Float32,
    `extra` Float32,
    `mta_tax` Float32,
    `tip_amount` Float32,
    `tolls_amount` Float32,
    `ehail_fee` Float32,
    `improvement_surcharge` Float32,
    `total_amount` Float32,
    `payment_type` Enum8('UNK' = 0, 'CSH' = 1, 'CRE' = 2, 'NOC' = 3, 'DIS' = 4),
    `trip_type` UInt8,
    `pickup` FixedString(25),
    `dropoff` FixedString(25),
    `cab_type` Enum8('yellow' = 1, 'green' = 2, 'uber' = 3),
    `pickup_nyct2010_gid` Int8,
    `pickup_ctlabel` Float32,
    `pickup_borocode` Int8,
    `pickup_ct2010` String,
    `pickup_boroct2010` String,
    `pickup_cdeligibil` String,
    `pickup_ntacode` FixedString(4),
    `pickup_ntaname` String,
    `pickup_puma` UInt16,
    `dropoff_nyct2010_gid` UInt8,
    `dropoff_ctlabel` Float32,
    `dropoff_borocode` UInt8,
    `dropoff_ct2010` String,
    `dropoff_boroct2010` String,
    `dropoff_cdeligibil` String,
    `dropoff_ntacode` FixedString(4),
    `dropoff_ntaname` String,
    `dropoff_puma` UInt16
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(pickup_date)
ORDER BY pickup_datetime;
clickhouse.ru-central1.internal :) show tables

SHOW TABLES

Query id: 97d5080f-3c67-489f-a501-28a1252001c1

┌─name──┐
│ trips │
└───────┘

1 row in set. Elapsed: 0.001 sec. 
`````

Загружаем данные

````
clickhouse.ru-central1.internal :) select count(1) from trips;

SELECT count(1)
FROM trips

Query id: 7a88a410-2633-4e88-93e9-21d0419391dd

┌─count()─┐
│ 1999657 │
└─────────┘

1 row in set. Elapsed: 0.001 sec. 
````

Выполним аналитические запросы:
````
SELECT
    avg(tip_amount) AS avg_tip,
    avg(fare_amount) AS avg_fare,
    avg(passenger_count) AS avg_passenger,
    count() AS count,
    truncate(dateDiff('second', pickup_datetime, dropoff_datetime) / 60) AS trip_minutes
FROM trips
WHERE trip_minutes > 0
GROUP BY trip_minutes
ORDER BY trip_minutes DESC

Query id: e7b773c4-c4fd-45a1-b2ae-3672ee7b5329

┌──────────────avg_tip─┬───────────avg_fare─┬──────avg_passenger─┬──count─┬─trip_minutes─┐
│   1.9600000381469727 │                  8 │                  1 │      1 │        27511 │
│                    0 │                 12 │                  2 │      1 │        27500 │
│    0.542166673981895 │ 19.716666666666665 │ 1.9166666666666667 │     60 │         1439 │
│    0.902499997522682 │ 11.270625001192093 │            1.95625 │    160 │         1438 │
│   0.9715789457909146 │ 13.646616541353383 │ 2.0526315789473686 │    133 │         1437 │
│   0.9682692398245518 │ 14.134615384615385 │  2.076923076923077 │    104 │         1436 │
│   1.1022105210705808 │ 13.778947368421052 │  2.042105263157895 │     95 │         1435 │
│   0.8419117699651157 │ 12.441176470588236 │ 2.3529411764705883 │     68 │         1434 │
│    0.894264710300109 │ 13.242647058823529 │  2.411764705882353 │     68 │         1433 │
│   0.8565151565002672 │ 15.992424242424242 │  2.212121212121212 │     66 │         1432 │
│   0.7527777817514207 │ 14.592592592592593 │ 1.8333333333333333 │     54 │         1431 │
│   1.5949999883770942 │            14.9375 │                2.4 │     40 │         1430 │
│   1.5668965393099292 │ 15.379310344827585 │ 1.6206896551724137 │     29 │         1429 │
│   1.1254838974245134 │ 12.451612903225806 │ 2.3225806451612905 │     31 │         1428 │
.........
│   0.5668389623567226 │  4.593474972149986 │ 1.6160541218501767 │  26311 │            1 │
└──────────────────────┴────────────────────┴────────────────────┴────────┴──────────────┘

618 rows in set. Elapsed: 0.044 sec. Processed 2.00 million rows, 33.99 MB (45.30 million rows/s., 770.12 MB/s.)

SELECT
    pickup_ntaname,
    toHour(pickup_datetime) as pickup_hour,
    SUM(1) AS pickups
FROM trips
WHERE pickup_ntaname = 'Belmont'
GROUP BY pickup_ntaname, pickup_hour
ORDER BY pickup_ntaname, pickup_hour

Query id: d5255d5b-5c40-4874-b174-6c723cfe3501

┌─pickup_ntaname─┬─pickup_hour─┬─pickups─┐
│ Belmont        │           7 │       1 │
│ Belmont        │           8 │       1 │
│ Belmont        │           9 │       1 │
│ Belmont        │          10 │       1 │
│ Belmont        │          11 │       1 │
│ Belmont        │          14 │       2 │
│ Belmont        │          15 │       1 │
│ Belmont        │          21 │       1 │
│ Belmont        │          23 │       1 │
└────────────────┴─────────────┴─────────┘

9 rows in set. Elapsed: 0.065 sec. Processed 2.00 million rows, 60.52 MB (30.60 million rows/s., 926.12 MB/s.)

SELECT
    pickup_datetime,
    dropoff_datetime,
    total_amount,
    pickup_nyct2010_gid,
    dropoff_nyct2010_gid,
    CASE
        WHEN dropoff_nyct2010_gid = 138 THEN 'LGA'
        WHEN dropoff_nyct2010_gid = 132 THEN 'JFK'
    END AS airport_code,
    EXTRACT(YEAR FROM pickup_datetime) AS year,
    EXTRACT(DAY FROM pickup_datetime) AS day,
    EXTRACT(HOUR FROM pickup_datetime) AS hour
FROM trips
WHERE dropoff_nyct2010_gid IN (132, 138)
ORDER BY pickup_datetime
LIMIT 20;

Query id: 09823c34-84b6-41ab-98f9-c22ccc5c36e4

┌─────pickup_datetime─┬────dropoff_datetime─┬─total_amount─┬─pickup_nyct2010_gid─┬─dropoff_nyct2010_gid─┬─airport_code─┬─year─┬─day─┬─hour─┐
│ 2015-07-01 00:04:14 │ 2015-07-01 00:15:29 │         13.3 │                 -34 │                  132 │ JFK          │ 2015 │   1 │    0 │
│ 2015-07-01 00:09:42 │ 2015-07-01 00:12:55 │          6.8 │                  50 │                  138 │ LGA          │ 2015 │   1 │    0 │
│ 2015-07-01 00:23:04 │ 2015-07-01 00:24:39 │          4.8 │                -125 │                  132 │ JFK          │ 2015 │   1 │    0 │
│ 2015-07-01 00:27:51 │ 2015-07-01 00:39:02 │        14.72 │                -101 │                  138 │ LGA          │ 2015 │   1 │    0 │
│ 2015-07-01 00:32:03 │ 2015-07-01 00:55:39 │        39.34 │                  48 │                  138 │ LGA          │ 2015 │   1 │    0 │
│ 2015-07-01 00:34:12 │ 2015-07-01 00:40:48 │         9.95 │                 -93 │                  132 │ JFK          │ 2015 │   1 │    0 │
│ 2015-07-01 00:38:26 │ 2015-07-01 00:49:00 │         13.3 │                 -11 │                  138 │ LGA          │ 2015 │   1 │    0 │
│ 2015-07-01 00:41:48 │ 2015-07-01 00:44:45 │          6.3 │                 -94 │                  132 │ JFK          │ 2015 │   1 │    0 │
│ 2015-07-01 01:06:18 │ 2015-07-01 01:14:43 │        11.76 │                  37 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:07:34 │ 2015-07-01 01:30:56 │         23.8 │                  42 │                  138 │ LGA          │ 2015 │   1 │    1 │
│ 2015-07-01 01:11:41 │ 2015-07-01 01:18:23 │        10.56 │                  14 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:18:13 │ 2015-07-01 01:27:49 │         11.3 │                  93 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:20:24 │ 2015-07-01 01:28:09 │          8.8 │                  79 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:44:03 │ 2015-07-01 02:09:52 │        69.34 │                   8 │                  138 │ LGA          │ 2015 │   1 │    1 │
│ 2015-07-01 01:47:32 │ 2015-07-01 01:56:00 │        12.25 │                  37 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:54:17 │ 2015-07-01 02:03:16 │        11.75 │                   4 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 01:58:07 │ 2015-07-01 02:05:27 │          9.8 │                  75 │                  132 │ JFK          │ 2015 │   1 │    1 │
│ 2015-07-01 02:08:04 │ 2015-07-01 02:18:01 │        12.96 │                -114 │                  138 │ LGA          │ 2015 │   1 │    2 │
│ 2015-07-01 02:10:00 │ 2015-07-01 02:14:56 │         9.96 │                 -31 │                  132 │ JFK          │ 2015 │   1 │    2 │
│ 2015-07-01 02:27:05 │ 2015-07-01 02:31:24 │          6.3 │                  21 │                  138 │ LGA          │ 2015 │   1 │    2 │
└─────────────────────┴─────────────────────┴──────────────┴─────────────────────┴──────────────────────┴──────────────┴──────┴─────┴──────┘

20 rows in set. Elapsed: 0.015 sec. Processed 392.45 thousand rows, 5.49 MB (25.59 million rows/s., 358.20 MB/s.)
`````
