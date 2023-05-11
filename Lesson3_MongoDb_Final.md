# Домашнее задание по уроку 3 Оптимизация производительности mongodb

### 1.Построить шардированный кластер из 3 кластерных нод( по 3 инстанса с репликацией) и с кластером конфига(3 инстанса);
Была выбрана следующая конфигурация проекта

| mongodbs | mongodbrs1          | mongodbrs2          | mongodbrs3           |
|----------|---------------------|---------------------|----------------------|
| mongos   | RScfg.node1         | RScfg.node2         | RScfg.node3          |
|          | RS1.node1 (primary) | RS1.node2           | RS1.node3            |
|          | RS2.node1           | RS2.node2 (primary) | RS2.node3            |
|          | RS3.node1           | RS3.node2           | RS3.node3  (primary) |

RScfg - реплика конфигурации шардирования
RS1, RS2, RS3 - реплики с данными

<b>Настройка кластера </b>
Для упрощения конфигурирования все инстансы создавались с параметром <i>--bind_ip_all</i> что не является секьюрным, но для тестового задания сделал такое допущение

><b>Общая настройка для всех серверов</b>
>
После включения всех ВМ правим /etc/hosts
```
Настраиваем /etc/hosts
51.250.97.147 mongodbrs3.ru-central1.internal mongodbrs3
158.160.18.174 mongodbrs1.ru-central1.internal mongodbrs1
84.201.154.0  mongodbrs2.ru-central1.internal mongodbrs2
158.160.30.107 mongodbs.ru-central1.internal mongodbs
```
>Создаем требуемые директории и запускаем инстансы

><b>mongodbrs1</b>

````
sudo mkdir /home/mongo && sudo mkdir /home/mongo/{dbc1,db1,db2,db3}
mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
````
><b>mongodbrs2</b>
````
sudo mkdir /home/mongo && sudo mkdir /home/mongo/{dbc1,db1,db2,db3}
mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
````
><b>mongodbrs3</b>
````
sudo mkdir /home/mongo && sudo mkdir /home/mongo/{dbc1,db1,db2,db3}
mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
````

>Собираем репликасеты

```
mongosh --port 27011
rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "mongodbrs1:27011"},{"_id" : 1, host : "mongodbrs2:27011"},{"_id" : 2, host : "mongodbrs3:27011"}]});
rs.status()
RS1 [direct: primary] test> rs.status()
{
  set: 'RS1',
  date: ISODate("2023-04-25T07:03:46.601Z"),
  myState: 1,
  term: Long("1"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-04-25T07:03:38.857Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-04-25T07:03:38.857Z"),
    lastDurableWallTime: ISODate("2023-04-25T07:03:38.857Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1682406193, i: 2 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate("2023-04-25T05:28:28.511Z"),
    electionTerm: Long("1"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1682400496, i: 1 }), t: Long("-1") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1682400496, i: 1 }), t: Long("-1") },
    numVotesNeeded: 2,
    priorityAtElection: 3,
    electionTimeoutMillis: Long("10000"),
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-04-25T05:28:28.595Z"),
    wMajorityWriteAvailabilityDate: ISODate("2023-04-25T05:28:29.730Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongodbrs1:27011',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 7028,
      optime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:03:38.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1682400508, i: 1 }),
      electionDate: ISODate("2023-04-25T05:28:28.000Z"),
      configVersion: 4,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongodbrs2:27011',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 5729,
      optime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:03:38.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:03:38.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      lastHeartbeat: ISODate("2023-04-25T07:03:46.421Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:03:46.421Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs1:27011',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'mongodbrs3:27011',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 1974,
      optime: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406218, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:03:38.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:03:38.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:03:38.857Z"),
      lastHeartbeat: ISODate("2023-04-25T07:03:46.500Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:03:45.439Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs2:27011',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1682406218, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1682406221, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1682406218, i: 1 })
}

mongosh --port 27012
rs.initiate({"_id" : "RS2", members : [{"_id" : 0, priority : 3, host : "mongodbrs2:27012"},{"_id" : 1, host : "mongodbrs3:27012"},{"_id" : 2, host : "mongodbrs1:27012"}]});
rs.status()
RS2 [direct: secondary] test> rs.status()
{
  set: 'RS2',
  date: ISODate("2023-04-25T07:04:21.588Z"),
  myState: 2,
  term: Long("1"),
  syncSourceHost: 'mongodbrs3:27012',
  syncSourceId: 1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-04-25T07:04:11.738Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-04-25T07:04:11.738Z"),
    lastDurableWallTime: ISODate("2023-04-25T07:04:11.738Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1682406241, i: 1 }),
  members: [
    {
      _id: 0,
      name: 'mongodbrs2:27012',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 1814,
      optime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:11.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:04:11.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      lastHeartbeat: ISODate("2023-04-25T07:04:21.104Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:04:19.987Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1682400591, i: 1 }),
      electionDate: ISODate("2023-04-25T05:29:51.000Z"),
      configVersion: 4,
      configTerm: 1
    },
    {
      _id: 1,
      name: 'mongodbrs3:27012',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 1814,
      optime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:11.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:04:11.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      lastHeartbeat: ISODate("2023-04-25T07:04:21.248Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:04:19.986Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs2:27012',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'mongodbrs1:27012',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 1816,
      optime: { ts: Timestamp({ t: 1682406251, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:11.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:11.738Z"),
      syncSourceHost: 'mongodbrs3:27012',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1682406251, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1682406251, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1682406251, i: 1 })
}

mongosh --port 27013
rs.initiate({"_id" : "RS3", members : [{"_id" : 0, priority : 3, host : "mongodbrs3:27013"},{"_id" : 1, host : "mongodbrs1:27013"},{"_id" : 2, host : "mongodbrs2:27013"}]});
rs.status()
RS3 [direct: secondary] test> rs.status()
{
  set: 'RS3',
  date: ISODate("2023-04-25T07:04:54.037Z"),
  myState: 2,
  term: Long("1"),
  syncSourceHost: 'mongodbrs3:27013',
  syncSourceId: 0,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1682406293, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-04-25T07:04:53.308Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1682406293, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1682406293, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1682406293, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-04-25T07:04:53.308Z"),
    lastDurableWallTime: ISODate("2023-04-25T07:04:53.308Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1682406273, i: 1 }),
  electionParticipantMetrics: {
    votedForCandidate: true,
    electionTerm: Long("1"),
    lastVoteDate: ISODate("2023-04-25T05:30:33.076Z"),
    electionCandidateMemberId: 0,
    voteReason: '',
    lastAppliedOpTimeAtElection: { ts: Timestamp({ t: 1682400622, i: 1 }), t: Long("-1") },
    maxAppliedOpTimeInSet: { ts: Timestamp({ t: 1682400622, i: 1 }), t: Long("-1") },
    priorityAtElection: 1,
    newTermStartDate: ISODate("2023-04-25T05:30:33.092Z"),
    newTermAppliedDate: ISODate("2023-04-25T05:30:34.640Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongodbrs3:27013',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 5670,
      optime: { ts: Timestamp({ t: 1682406283, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406283, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:43.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:04:43.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:43.308Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:43.308Z"),
      lastHeartbeat: ISODate("2023-04-25T07:04:52.467Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:04:52.627Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1682400633, i: 1 }),
      electionDate: ISODate("2023-04-25T05:30:33.000Z"),
      configVersion: 4,
      configTerm: 1
    },
    {
      _id: 1,
      name: 'mongodbrs1:27013',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 6983,
      optime: { ts: Timestamp({ t: 1682406293, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:53.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:53.308Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:53.308Z"),
      syncSourceHost: 'mongodbrs3:27013',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 2,
      name: 'mongodbrs2:27013',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 1343,
      optime: { ts: Timestamp({ t: 1682406283, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406283, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:04:43.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:04:43.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:04:53.308Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:04:53.308Z"),
      lastHeartbeat: ISODate("2023-04-25T07:04:52.435Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:04:53.448Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs1:27013',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 1
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1682406293, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1682406293, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1682406293, i: 1 })
}

mongosh --port 27001
rs.initiate({"_id" : "RScfg", members : [{"_id" : 0, host : "mongodbrs1:27001"},{"_id" : 1, host : "mongodbrs2:27001"},{"_id" : 2, host : "mongodbrs3:27001"}]});
rs.status()
RScfg [direct: secondary] test> rs.status()
{
  set: 'RScfg',
  date: ISODate("2023-04-25T07:02:32.445Z"),
  myState: 2,
  term: Long("1"),
  syncSourceHost: 'mongodbrs2:27001',
  syncSourceId: 1,
  configsvr: true,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1682406151, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-04-25T07:02:31.464Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1682406151, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1682406151, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1682406151, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-04-25T07:02:31.464Z"),
    lastDurableWallTime: ISODate("2023-04-25T07:02:31.464Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1682406111, i: 1 }),
  electionParticipantMetrics: {
    votedForCandidate: true,
    electionTerm: Long("1"),
    lastVoteDate: ISODate("2023-04-25T05:49:46.866Z"),
    electionCandidateMemberId: 1,
    voteReason: '',
    lastAppliedOpTimeAtElection: { ts: Timestamp({ t: 1682401775, i: 1 }), t: Long("-1") },
    maxAppliedOpTimeInSet: { ts: Timestamp({ t: 1682401775, i: 1 }), t: Long("-1") },
    priorityAtElection: 1,
    newTermStartDate: ISODate("2023-04-25T05:49:46.892Z"),
    newTermAppliedDate: ISODate("2023-04-25T05:49:47.927Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongodbrs1:27001',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 7160,
      optime: { ts: Timestamp({ t: 1682406151, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:02:31.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:02:31.464Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:02:31.464Z"),
      syncSourceHost: 'mongodbrs2:27001',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongodbrs2:27001',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 4376,
      optime: { ts: Timestamp({ t: 1682406150, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406150, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:02:30.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:02:30.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:02:30.464Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:02:30.464Z"),
      lastHeartbeat: ISODate("2023-04-25T07:02:31.341Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:02:31.733Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1682401786, i: 1 }),
      electionDate: ISODate("2023-04-25T05:49:46.000Z"),
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'mongodbrs3:27001',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 4376,
      optime: { ts: Timestamp({ t: 1682406150, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1682406150, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-04-25T07:02:30.000Z"),
      optimeDurableDate: ISODate("2023-04-25T07:02:30.000Z"),
      lastAppliedWallTime: ISODate("2023-04-25T07:02:30.464Z"),
      lastDurableWallTime: ISODate("2023-04-25T07:02:30.464Z"),
      lastHeartbeat: ISODate("2023-04-25T07:02:31.433Z"),
      lastHeartbeatRecv: ISODate("2023-04-25T07:02:31.939Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs2:27001',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1682406151, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1682406151, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1682406151, i: 1 })
}
```

>Запускаем mongos

<b>mongodbs</b>
```
sudo mkdir /home/mongo && sudo mkdir /home/mongo/dbs1
mongos --configdb RScfg/mongodbrs1:27001,mongodbrs2:27001,mongodbrs3:27001 --port 27000 --fork --logpath /home/mongo/dbs1/dbs.log --pidfilepath /home/mongo/dbs1/dbs1.pid --bind_ip_all
```

>Собираем шарды
````
mongosh --port 27000
sh.addShard("RS1/mongodbrs1:27011,mongodbrs2:27011,mongodbrs3:27011")
sh.addShard("RS2/mongodbrs1:27012,mongodbrs2:27012,mongodbrs3:27012")
sh.addShard("RS3/mongodbrs1:27013,mongodbrs2:27013,mongodbrs3:27013")
sh.status()

[direct: mongos] test> sh.status()
shardingVersion
{
  _id: 1,
  minCompatibleVersion: 5,
  currentVersion: 6,
  clusterId: ObjectId("644769fb72a6276f647e844e")
}
---
shards
[
  {
    _id: 'RS1',
    host: 'RS1/mongodbrs1:27011,mongodbrs2:27011,mongodbrs3:27011',
    state: 1,
    topologyTime: Timestamp({ t: 1682404277, i: 1 })
  },
  {
    _id: 'RS2',
    host: 'RS2/mongodbrs1:27012,mongodbrs2:27012,mongodbrs3:27012',
    state: 1,
    topologyTime: Timestamp({ t: 1682404475, i: 4 })
  },
  {
    _id: 'RS3',
    host: 'RS3/mongodbrs1:27013,mongodbrs2:27013,mongodbrs3:27013',
    state: 1,
    topologyTime: Timestamp({ t: 1682404974, i: 6 })
  }
]
---
active mongoses
[ { '6.0.5': 1 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently running': 'no',
  'Currently enabled': 'yes',
  'Failed balancer rounds in last 5 attempts': 0,
  'Migration Results for the last 24 hours': 'No recent migrations'
}
---
databases
[
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS1', nChunks: 1024 } ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  }
]
````



### 2. Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами;

> Выполняем добавление новой БД и коллекции со случайными данными на mongodbs
```
[direct: mongos] admin> use bank
switched to db bank
[direct: mongos] bank> sh.enableSharding("bank")
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683784601, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683784601, i: 1 })
}
[direct: mongos] bank> use config
switched to db config
[direct: mongos] config> db.settings.updateOne(
...    { _id: "chunksize" },
...    { $set: { _id: "chunksize", value: 1 } },
...    { upsert: true }
... )
{
  acknowledged: true,
  insertedId: 'chunksize',
  matchedCount: 0,
  modifiedCount: 0,
  upsertedCount: 1
}
[direct: mongos] config> use bank
switched to db bank
[direct: mongos] bank> for (var i=0; i<50000; i++) { db.tickets.insert({name: "Max ammout of cost tickets", amount: Math.random()*100}) }
DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
{
  acknowledged: true,
  insertedIds: { '0': ObjectId("645c85281f5b841aa1541a0f") }
}
[direct: mongos] bank> db.tickets.createIndex({amount: 1})
amount_1
 
direct: mongos] bank> use admin
switched to db admin
[direct: mongos] admin> db.runCommand({shardCollection: "bank.tickets", key: {amount: 1}})
{
  collectionsharded: 'bank.tickets',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683785163, i: 10 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683785163, i: 6 })
}
sh.status()
...
collections: {
      'bank.tickets': {
        shardKey: { amount: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS2', nChunks: 1 }, { shard: 'RS3', nChunks: 1 } ],
        chunks: [
          { min: { amount: MinKey() }, max: { amount: 28.141677578987622 }, 'on shard': 'RS3', 'last modified': Timestamp({ t: 2, i: 0 }) },
          { min: { amount: 28.141677578987622 }, max: { amount: MaxKey() }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 1 }) }
        ],
        tags: []
      }
    }
 
[direct: mongos] admin> sh.balancerCollectionStatus("bank.tickets")
{
  chunkSize: 1,
  balancerCompliant: true,
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683785269, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683785269, i: 1 })
}
```
> Разделим чанки
```
[direct: mongos] admin> sh.splitFind( "bank.tickets", { "amount": "50" } )
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683785436, i: 5 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683785436, i: 4 })
}

[direct: mongos] admin> sh.status()
...
collections: {
      'bank.tickets': {
        shardKey: { amount: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS2', nChunks: 3 }, { shard: 'RS3', nChunks: 1 } ],
        chunks: [
          { min: { amount: MinKey() }, max: { amount: 28.141677578987622 }, 'on shard': 'RS3', 'last modified': Timestamp({ t: 2, i: 0 }) },
          { min: { amount: 28.141677578987622 }, max: { amount: 64.19224381416981 }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 2 }) },
          { min: { amount: 64.19224381416981 }, max: { amount: 82.17083678216588 }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 4 }) },
          { min: { amount: 82.17083678216588 }, max: { amount: MaxKey() }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 5 }) }
        ],
        tags: []
      }
    }
...
```
> Настраиваем балансер
````
[direct: mongos] admin> sh.getBalancerState()
true
[direct: mongos] admin> sh.isBalancerRunning()
{
  mode: 'full',
  inBalancerRound: false,
  numBalancerRounds: Long("260"),
  term: Long("3"),
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683785806, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683785806, i: 1 })
}

[direct: mongos] admin> use config
switched to db config

[direct: mongos] config> sh.startBalancer()
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683785833, i: 3 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683785833, i: 3 })
}
````
> Настроим окно активности
````
[direct: mongos] config> db.settings.updateOne(
...    { _id: "balancer" },
...    { $set: { activeWindow : { start : "00:00", stop : "23:59" } } },
...    { upsert: true }
... )
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
[direct: mongos] config> sh.status()
...
    collections: {
      'bank.tickets': {
        shardKey: { amount: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'RS2', nChunks: 3 }, { shard: 'RS3', nChunks: 1 } ],
        chunks: [
          { min: { amount: MinKey() }, max: { amount: 28.141677578987622 }, 'on shard': 'RS3', 'last modified': Timestamp({ t: 2, i: 0 }) },
          { min: { amount: 28.141677578987622 }, max: { amount: 64.19224381416981 }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 2 }) },
          { min: { amount: 64.19224381416981 }, max: { amount: 82.17083678216588 }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 4 }) },
          { min: { amount: 82.17083678216588 }, max: { amount: MaxKey() }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 2, i: 5 }) }
        ],
        tags: []
      }
    }
  }
  ...
 ````
 
### 3. Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.

> Остановим праймари для репликасета RS1
````
RS1 [direct: primary] bank> rs.status()
{
  set: 'RS1',
  date: ISODate("2023-05-11T06:29:10.445Z"),
  myState: 1,
  term: Long("4"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
    lastCommittedWallTime: ISODate("2023-05-11T06:29:08.557Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
    appliedOpTime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
    durableOpTime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
    lastAppliedWallTime: ISODate("2023-05-11T06:29:08.557Z"),
    lastDurableWallTime: ISODate("2023-05-11T06:29:08.557Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1683786498, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'priorityTakeover',
    lastElectionDate: ISODate("2023-05-11T05:32:58.406Z"),
    electionTerm: Long("4"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1683783176, i: 1 }), t: Long("3") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1683783176, i: 1 }), t: Long("3") },
    numVotesNeeded: 2,
    priorityAtElection: 3,
    electionTimeoutMillis: Long("10000"),
    priorPrimaryMemberId: 1,
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-05-11T05:32:58.417Z"),
    wMajorityWriteAvailabilityDate: ISODate("2023-05-11T05:32:59.423Z")
  },
  electionParticipantMetrics: {
    votedForCandidate: true,
    electionTerm: Long("3"),
    lastVoteDate: ISODate("2023-05-11T05:32:46.895Z"),
    electionCandidateMemberId: 1,
    voteReason: '',
    lastAppliedOpTimeAtElection: { ts: Timestamp({ t: 1682406477, i: 1 }), t: Long("2") },
    maxAppliedOpTimeInSet: { ts: Timestamp({ t: 1682406477, i: 1 }), t: Long("2") },
    priorityAtElection: 3
  },
  members: [
    {
      _id: 0,
      name: 'mongodbrs1:27011',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 3466,
      optime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
      optimeDate: ISODate("2023-05-11T06:29:08.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1683783178, i: 2 }),
      electionDate: ISODate("2023-05-11T05:32:58.000Z"),
      configVersion: 4,
      configTerm: 4,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongodbrs2:27011',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 3393,
      optime: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
      optimeDurable: { ts: Timestamp({ t: 1683786548, i: 1 }), t: Long("4") },
      optimeDate: ISODate("2023-05-11T06:29:08.000Z"),
      optimeDurableDate: ISODate("2023-05-11T06:29:08.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      lastHeartbeat: ISODate("2023-05-11T06:29:09.657Z"),
      lastHeartbeatRecv: ISODate("2023-05-11T06:29:09.316Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs1:27011',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 4,
      configTerm: 4
    },
    {
      _id: 2,
      name: 'mongodbrs3:27011',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 3287,
      optime: { ts: Timestamp({ t: 1683786538, i: 2 }), t: Long("4") },
      optimeDurable: { ts: Timestamp({ t: 1683786538, i: 2 }), t: Long("4") },
      optimeDate: ISODate("2023-05-11T06:28:58.000Z"),
      optimeDurableDate: ISODate("2023-05-11T06:28:58.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:29:08.557Z"),
      lastHeartbeat: ISODate("2023-05-11T06:29:08.558Z"),
      lastHeartbeatRecv: ISODate("2023-05-11T06:29:09.723Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs2:27011',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 4
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1683786548, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683786548, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683786548, i: 1 })
}

root@mongodbrs2:~# mongosh --port 27011
RS1 [direct: primary] test> rs.status()
{
  set: 'RS1',
  date: ISODate("2023-05-11T06:30:55.604Z"),
  myState: 1,
  term: Long("5"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
    lastCommittedWallTime: ISODate("2023-05-11T06:30:52.399Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
    appliedOpTime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
    durableOpTime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
    lastAppliedWallTime: ISODate("2023-05-11T06:30:52.399Z"),
    lastDurableWallTime: ISODate("2023-05-11T06:30:52.399Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1683786632, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'stepUpRequestSkipDryRun',
    lastElectionDate: ISODate("2023-05-11T06:30:02.347Z"),
    electionTerm: Long("5"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1683786598, i: 2 }), t: Long("4") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1683786598, i: 2 }), t: Long("4") },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long("10000"),
    priorPrimaryMemberId: 0,
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-05-11T06:30:02.383Z"),
    wMajorityWriteAvailabilityDate: ISODate("2023-05-11T06:30:02.408Z")
  },
  electionParticipantMetrics: {
    votedForCandidate: true,
    electionTerm: Long("4"),
    lastVoteDate: ISODate("2023-05-11T05:32:58.408Z"),
    electionCandidateMemberId: 0,
    voteReason: '',
    lastAppliedOpTimeAtElection: { ts: Timestamp({ t: 1683783176, i: 1 }), t: Long("3") },
    maxAppliedOpTimeInSet: { ts: Timestamp({ t: 1683783176, i: 1 }), t: Long("3") },
    priorityAtElection: 1
  },
  members: [
    {
      _id: 0,
      name: 'mongodbrs1:27011',
      health: 0,
      state: 8,
      stateStr: '(not reachable/healthy)',
      uptime: 0,
      optime: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
      optimeDurable: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
      optimeDate: ISODate("1970-01-01T00:00:00.000Z"),
      optimeDurableDate: ISODate("1970-01-01T00:00:00.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:30:12.398Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:30:12.398Z"),
      lastHeartbeat: ISODate("2023-05-11T06:30:52.437Z"),
      lastHeartbeatRecv: ISODate("2023-05-11T06:30:15.490Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: "Couldn't get a connection within the time limit",
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 5
    },
    {
      _id: 1,
      name: 'mongodbrs2:27011',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 3502,
      optime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
      optimeDate: ISODate("2023-05-11T06:30:52.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:30:52.399Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:30:52.399Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1683786602, i: 1 }),
      electionDate: ISODate("2023-05-11T06:30:02.000Z"),
      configVersion: 4,
      configTerm: 5,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 2,
      name: 'mongodbrs3:27011',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 3394,
      optime: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
      optimeDurable: { ts: Timestamp({ t: 1683786652, i: 1 }), t: Long("5") },
      optimeDate: ISODate("2023-05-11T06:30:52.000Z"),
      optimeDurableDate: ISODate("2023-05-11T06:30:52.000Z"),
      lastAppliedWallTime: ISODate("2023-05-11T06:30:52.399Z"),
      lastDurableWallTime: ISODate("2023-05-11T06:30:52.399Z"),
      lastHeartbeat: ISODate("2023-05-11T06:30:54.423Z"),
      lastHeartbeatRecv: ISODate("2023-05-11T06:30:54.422Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongodbrs2:27011',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 4,
      configTerm: 5
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1683786652, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1683786652, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1683786652, i: 1 })
}

````

Видим,что праймари переехал на mongodbrs2, а mongodbrs1 стал **not reachable/healthy**


### 4. Настроить аутентификацию и многоролевой доступ
> Создадим отдельный инстанс и проведем настройку на нем

````
root@mongodbs:~# sudo mkdir /home/mongo/db15 && sudo chmod 777 /home/mongo/db15
root@mongodbs:~# mongod --dbpath /home/mongo/db15 --port 27005 --fork --logpath /home/mongo/db15/db5.log --pidfilepath /home/mongo/db15/db5.pid --bind_ip_all
about to fork child process, waiting until server is ready for connections.
forked process: 1488
child process started successfully, parent exiting
root@mongodbs:~# mongo --port 27005
> db.createRole(
...     {      
...      role: "superRoot",      
...      privileges:[
...         { resource: {anyResource:true}, actions: ["anyAction"]}
...      ],      
...      roles:[] 
...     }
... )
{
        "role" : "superRoot",
        "privileges" : [
                {
                        "resource" : {
                                "anyResource" : true
                        },
                        "actions" : [
                                "anyAction"
                        ]
                }
        ],
        "roles" : [ ]
}

> db.createUser({      
...      user: "companyDBA",      
...      pwd: "EWqeeFpUt",      
...      roles: ["superRoot"],
...      passwordDigestor:"server"
... })
Successfully added user: {
        "user" : "companyDBA",
        "roles" : [
                "superRoot"
        ],
        "passwordDigestor" : "server"
}


> db.system.roles.find()
{ "_id" : "admin.superRoot", "role" : "superRoot", "db" : "admin", "privileges" : [ { "resource" : { "anyResource" : true }, "actions" : [ "anyAction" ] } ], "roles" : [ ] }

> db.system.users.find()
{ "_id" : "admin.companyDBA", "userId" : UUID("8acef635-2743-4875-9bd9-f90e02fd9e93"), "user" : "companyDBA", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "tueG/w59wwUqACVzXqRUNA==", "storedKey" : "GV2+pk/sboFgzStPJ21+Mo7K9MA=", "serverKey" : "FgUhs3kClfaRdlLNVteD9f7i8/Y=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "uBSaYpp1FNid6XQesm2CkqaIDVNMOXco52SqKw==", "storedKey" : "skVKJKpUUG74Yai4FwZV/zAPDoz1zI5DrPu8hx4Ka6w=", "serverKey" : "Q51vN3OVIXjHKKMqtSxFDKT0LKNrStwIn1yUJKErppg=" } }, "roles" : [ { "role" : "superRoot", "db" : "admin" } ] }
````
> Запустим с включенным режимом аутентификации
````
mongod --dbpath /home/mongo/db15 --port 27005 --auth --fork --logpath /home/mongo/db15/db5.log --pidfilepath /home/mongo/db15/db5.pid

root@mongodbrs1:/home/mongo/db15# mongo --port 27006
MongoDB shell version v3.6.8
connecting to: mongodb://127.0.0.1:27005/
Implicit session: session { "id" : UUID("758fa844-2c57-4cc0-8728-4715dcadc5d1") }
MongoDB server version: 3.6.8
> db.system.users.find()
Error: error: {
        "ok" : 0,
        "errmsg" : "there are no users authenticated",
        "code" : 13,
        "codeName" : "Unauthorized"
}

root@mongodbrs1:/home/mongo/db15# mongo --port 27006 -u companyDBA -p EWqeeFpUt --authenticationDatabase "admin"
MongoDB shell version v3.6.8
connecting to: mongodb://127.0.0.1:27006/
Implicit session: session { "id" : UUID("84d21849-528b-45fe-9f9e-638a4754864d") }
MongoDB server version: 3.6.8
Server has startup warnings: 
2023-05-11T07:05:18.523+0000 I STORAGE  [initandlisten] 
2023-05-11T07:05:18.523+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2023-05-11T07:05:18.523+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2023-05-11T07:05:19.633+0000 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2023-05-11T07:05:19.633+0000 I CONTROL  [initandlisten] 
> show databases
admin   0.000GB
config  0.000GB
local   0.000GB


````
