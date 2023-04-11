# Домашнее задание по уроку 2 Основы MongoDb
## установить MongoDB одним из способов: ВМ, докер;

произведена установка MongoDb версии 6.0 на ВМ в облаке yandex

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
