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



### Добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами;
### Поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.
### Настроить аутентификацию и многоролевой доступ;
