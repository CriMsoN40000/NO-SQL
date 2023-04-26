# Домашнее задание по уроку Couchbase
## Развернуть кластер Couchbase
Кластер разворачивается в облаке Yandex Cloud. 
Подготовлены 4 ВМ
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

## Создать БД, наполнить небольшими тестовыми данными
##Проверить отказоустойчивость
