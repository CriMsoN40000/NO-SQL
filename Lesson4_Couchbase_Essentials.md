# Домашнее задание по уроку Couchbase
## Развернуть кластер Couchbase
Кластер разворачивается в облаке Yandex Cloud. 
> Подготовлены 4 ВМ
- cb1.ru-central1.internal
- cb2.ru-central1.internal
- cb3.ru-central1.internal
- cb4.ru-central1.internal

Произведена установка дистрибутива CouchBase 7.1
```
root@cb1:~# systemctl list-units --type service | grep couchbase
couchbase-server.service             loaded active running Couchbase Server
```

Создан кластер albcb, в который добавлены 4 ноды
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/cluster_ov.png"/> 

## Создать БД, наполнить небольшими тестовыми данными
Создаем тестовый бакет и наполняем его тестовыми данными
Переходим в раздел меню **Buckets**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_1.png"/>

Добавляем демо бакет 
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_2.png"/>

Видим, что под данный бакет использовано **800MiB**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_3.png"/>

Всего документов в бакете **586**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_5.png"/>

Выполним тестовые запросы
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_4.png"/>
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/bucket_6.png"/>


## Проверить отказоустойчивость
Выключим автоматический Failover
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_01.png"/>

Выключаем ноду **cb3** и видим сообщение, что сервер недоступен
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_02.png"/>

При этом запросы перестали выполняться, требуется активировать процедуру Failover, чтообы получить доступ к репликам
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_03.png"/>

Через 5 минут  поднимаем сервер **cb3**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_04.png"/>

Сервер автоматически добавляется в конфигурацию и запросы начинают работать
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_05.png"/>

Снова выключаем ноду **cb3** и видим сообщение, что сервер недоступен. Запросы остановились
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_06.png"/>

Запускаем **failover**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_07.png"/>
Видим, что сервер cb3 пропал из конфигурации

После запускаем **Rebalance**
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_08.png"/>

И видим, что запросы заработали
<img src="https://github.com/CriMsoN40000/NO-SQL/blob/main/failover_09.png"/>
