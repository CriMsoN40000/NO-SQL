# Домашнее задание по уроку 2 Основы MongoDb
## установить MongoDB одним из способов: ВМ, докер;

произведена установка MongoDb версии 6.0 на ВМ в облаке yandex
```
root@mongodb:~# systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-04-10 17:11:18 UTC; 12h ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 1867 (mongod)
     Memory: 170.0M
     CGroup: /system.slice/mongod.service
             └─1867 /usr/bin/mongod --config /etc/mongod.conf

Apr 10 17:11:18 mongodb systemd[1]: Started MongoDB Database Server.
```
````
db> db.hello()
{
  isWritablePrimary: true,
  topologyVersion: {
    processId: ObjectId("64344336ae66ca28ef943fd2"),
    counter: Long("0")
  },
  maxBsonObjectSize: 16777216,
  maxMessageSizeBytes: 48000000,
  maxWriteBatchSize: 100000,
  localTime: ISODate("2023-04-11T06:02:31.954Z"),
  logicalSessionTimeoutMinutes: 30,
  connectionId: 39,
  minWireVersion: 0,
  maxWireVersion: 17,
  readOnly: false,
  ok: 1
}
````


## Заполнить данными
Для импорта коллекции использовал команду
`````
root@mongodb:~# mongoimport --type csv -d otus -c less2 --headerline /home/alext/JEOPARDY_CSV.csv -u root --authenticationDatabase admin
Enter password for mongo user:

2023-04-11T06:21:43.192+0000    connected to: mongodb://localhost/
2023-04-11T06:21:46.192+0000    [###########.............] otus.less2   15.6MB/33.2MB (47.0%)
2023-04-11T06:21:49.193+0000    [#######################.] otus.less2   32.6MB/33.2MB (98.2%)
2023-04-11T06:21:49.345+0000    [########################] otus.less2   33.2MB/33.2MB (100.0%)
2023-04-11T06:21:49.345+0000    216930 document(s) imported successfully. 0 document(s) failed to import.

otus> show collections
less2

otus> db.less2.countDocuments()
216930
`````


