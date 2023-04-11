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
```


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

otus> db.less2.find().limit(10)
[
  {
    _id: ObjectId("6434fc771f0e5a2054b16c3f"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: "ESPN's TOP 10 ALL-TIME ATHLETES",
    Value: '$200',
    Question: 'No. 2: 1912 Olympian; football star at Carlisle Indian School; 6 MLB seasons with the Reds, Giants & Braves',
    Answer: 'Jim Thorpe'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c40"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'EVERYBODY TALKS ABOUT IT...',
    Value: '$200',
    Question: 'The city of Yuma in this state has a record average of 4,055 hours of sunshine each year',
    Answer: 'Arizona'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c41"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'HISTORY',
    Value: '$200',
    Question: "For the last 8 years of his life, Galileo was under house arrest for espousing this man's theory",
    Answer: 'Copernicus'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c42"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'THE COMPANY LINE',
    Value: '$200',
    Question: 'In 1963, live on "The Art Linkletter Show", this company served its billionth burger',
    Answer: "McDonald's"
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c43"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'EPITAPHS & TRIBUTES',
    Value: '$200',
    Question: 'Signer of the Dec. of Indep., framer of the Constitution of Mass., second President of the United States',
    Answer: 'John Adams'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c44"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'HISTORY',
    Value: '$400',
    Question: "Built in 312 B.C. to link Rome & the South of Italy, it's still in use today",
    Answer: 'the Appian Way'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c45"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: "ESPN's TOP 10 ALL-TIME ATHLETES",
    Value: '$400',
    Question: 'No. 8: 30 steals for the Birmingham Barons; 2,306 steals for the Bulls',
    Answer: 'Michael Jordan'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c46"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'EVERYBODY TALKS ABOUT IT...',
    Value: '$400',
    Question: 'In the winter of 1971-72, a record 1,122 inches of snow fell at Rainier Paradise Ranger Station in this state',
    Answer: 'Washington'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c47"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'THE COMPANY LINE',
    Value: '$400',
    Question: 'This housewares store was named for the packaging its merchandise came in & was first displayed on',
    Answer: 'Crate & Barrel'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c48"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: '3-LETTER WORDS',
    Value: '$200',
    Question: 'In the title of an Aesop fable, this insect shared billing with a grasshopper',
    Answer: 'the ant'
  }
]
`````

## Написать несколько запросов на выборку и обновление данных

```
otus> db.less2.find({"Round": 'Jeopardy!'}).limit(3)
[
  {
    _id: ObjectId("6434fc771f0e5a2054b16c3f"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: "ESPN's TOP 10 ALL-TIME ATHLETES",
    Value: '$200',
    Question: 'No. 2: 1912 Olympian; football star at Carlisle Indian School; 6 MLB seasons with the Reds, Giants & Braves',
    Answer: 'Jim Thorpe'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c40"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'EVERYBODY TALKS ABOUT IT...',
    Value: '$200',
    Question: 'The city of Yuma in this state has a record average of 4,055 hours of sunshine each year',
    Answer: 'Arizona'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c41"),
    'Show Number': 4680,
    'Air Date': '2004-12-31',
    Round: 'Jeopardy!',
    Category: 'HISTORY',
    Value: '$200',
    Question: "For the last 8 years of his life, Galileo was under house arrest for espousing this man's theory",
    Answer: 'Copernicus'
  }
]

otus> db.less2.find({"Round": 'Jeopardy!'}, {"Question":1, "Answer": 1}).limit(3)
[
  {
    _id: ObjectId("6434fc771f0e5a2054b16c3f"),
    Question: 'No. 2: 1912 Olympian; football star at Carlisle Indian School; 6 MLB seasons with the Reds, Giants & Braves',
    Answer: 'Jim Thorpe'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c40"),
    Question: 'The city of Yuma in this state has a record average of 4,055 hours of sunshine each year',
    Answer: 'Arizona'
  },
  {
    _id: ObjectId("6434fc771f0e5a2054b16c41"),
    Question: "For the last 8 years of his life, Galileo was under house arrest for espousing this man's theory",
    Answer: 'Copernicus'
  }
]

otus> db.less2.find({"Round": 'Jeopardy!'}, {"Question":1, "Answer": 1, "_id": 0}).limit(1)
[
  {
    Question: 'No. 2: 1912 Olympian; football star at Carlisle Indian School; 6 MLB seasons with the Reds, Giants & Braves',
    Answer: 'Jim Thorpe'
  }
]

otus> db.less2.find({"Value": {$in: ["$100"]}}, {"Question":1, "Answer": 1, "_id": 0}).limit(4)
[
  {
    Question: `Prime Minister Tony Blair dubbed her "The People's Princess"`,
    Answer: 'Princess Diana'
  },
  {
    Question: 'Once Tommy Mullaney on "L.A. Law", John Spencer now plays White House chief of staff Leo McGarry on this series',
    Answer: 'The West Wing'
  },
  {
    Question: "The Cinderella Castle Mystery Tour is a highlight of this Asian city's Disneyland",
    Answer: 'Tokyo'
  },
  {
    Question: 'This punk rock hitmaker heard here has had numerous hits on both sides of the Atlantic',
    Answer: 'Billy Idol'
  }
]
```

## создать индексы и сравнить производительность.

Запрос до создания индекса - <b> 11 ms </b>
````
otus> db.less2.find({"Value": {$in: ["$200"]}})
{
 "stage": "COLLSCAN",
 "filter": {
  "Value": {
   "$eq": "$200"
  }
 },
 "nReturned": 30455,
 "executionTimeMillisEstimate": 11,
 "works": 216932,
 "advanced": 30455,
 "needTime": 186476,
 "needYield": 0,
 "saveState": 216,
 "restoreState": 216,
 "isEOF": 1,
 "direction": "forward",
 "docsExamined": 216930
}
}
`````

Cоздаем индекс на поле <b>Value</b>
````
otus> db.less2.createIndex({"Value": "text"})
Value_text
````


После создания индекса запрос выполнился за 8ms

````
{
 "stage": "COLLSCAN",
 "filter": {
  "Value": {
   "$eq": "$200"
  }
 },
 "nReturned": 30455,
 "executionTimeMillisEstimate": 7,
 "works": 216932,
 "advanced": 30455,
 "needTime": 186476,
 "needYield": 0,
 "saveState": 216,
 "restoreState": 216,
 "isEOF": 1,
 "direction": "forward",
 "docsExamined": 216930
}
