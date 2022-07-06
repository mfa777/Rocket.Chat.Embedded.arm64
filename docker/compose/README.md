# docker compose to start an RC server

## Quick start

First, you need to make sure you have the `rc`  docker image build and available in your docker registry.

See the `README.md` in  the `rocketchat`` directory on how to build the image.

### start up mongo server

Once you have the images built, make sure you have a `data` directory - then you can startup the mongo server:

```
sudo docker compose up -d mongo
```

The mongo server will use the `data` directory to store the rocketchat data.   You can take a look in the directory to see that it is populated.

Examine the logs to see that the server is listening to connections:

```
sudo docker logs mongo
```

### initialize mongo replicaset

Next, you need to do this ONE TIME ONLY - to initialize the replicaset in mongo.  This turns the mongo server into a single primary  replicaset node.

```
sudo docker compose run --rm  mongo-init-replica
```

You should see output similar to:

```
compose_mongo_1 is up-to-date
Creating compose_mongo-init-replica_1 ... done
Attaching to compose_mongo-init-replica_1
mongo-init-replica_1  | MongoDB shell version: 3.2.15
mongo-init-replica_1  | connecting to: mongo/rocketchat
mongo-init-replica_1  | { "ok" : 1 }
compose_mongo-init-replica_1 exited with code 0
```
Note the `{ "ok" : 1 }` responose.

You can also check that the mongo server indeed has started up as a PRIMARY node via the logs:

```
sudo docker logs mongo
```

Look for similar message to ones below:

```
2019-02-22T07:54:59.517+0000 I REPL     [ReplicationExecutor] This node is localhost:27017 in the config
2019-02-22T07:54:59.517+0000 I REPL     [ReplicationExecutor] transition to STARTUP2
2019-02-22T07:54:59.517+0000 I REPL     [conn1] Starting replication applier threads
2019-02-22T07:54:59.519+0000 I REPL     [ReplicationExecutor] transition to RECOVERING
2019-02-22T07:54:59.520+0000 I REPL     [ReplicationExecutor] transition to SECONDARY
2019-02-22T07:54:59.520+0000 I REPL     [ReplicationExecutor] conducting a dry run election to see if we could be elected
2019-02-22T07:54:59.520+0000 I REPL     [ReplicationExecutor] dry election run succeeded, running for election
2019-02-22T07:54:59.524+0000 I REPL     [ReplicationExecutor] election succeeded, assuming primary role in term 1
2019-02-22T07:54:59.524+0000 I REPL     [ReplicationExecutor] transition to PRIMARY
2019-02-22T07:54:59.531+0000 I NETWORK  [conn1] end connection 172.18.0.3:56380 (0 connections now open)
```
Note that mongo node is now a PRIMARY node. Your mongo server is now up and running as a single node PRIMARY replicaset.

### change Site URL

Run mongo shell within docker container:

```
sudo docker compose exec mongo mongo
```

In mongo shell, run: `use rocketchat` -> `db.rocketchat_settings.update({"_id" : "Site_Url"},{$set:{value:"http://NEW_URL"}})`

### start the rocketchat server

With the mongo server up and running, you can now start the rocketchat server.  Use the command:

```
sudo docker compose up -d rocketchat
```

Starting the rocketchat server on a Raspberry Pi or 32 bit ARM SoC board will probably take a minute or two, so please be patient.

Meanwhile, you can check the logs to see the progress:

```
sudo docker compose logs rocketchat -f
```

Keep checking the logs until you see the `SERVER RUNNING` message box similar to the one below:


```
Updating process.env.MAIL_URL
Using GridFS for custom sounds storage
Using GridFS for custom emoji storage
Browserslist: caniuse-lite is outdated. Please run next command `npm update`
➔ System ➔ startup
➔ +----------------------------------------------+
➔ |                SERVER RUNNING                |
➔ +----------------------------------------------+
➔ |                                              |
➔ |  Rocket.Chat Version: 2.4.9                  |
➔ |       NodeJS Version: 8.17.0 - arm64         |
➔ |      MongoDB Version: 4.0.12                 |
➔ |       MongoDB Engine: wiredTiger             |
➔ |             Platform: linux                  |
➔ |         Process Port: 3000                   |
➔ |             Site URL: http://localhost:3000  |
➔ |     ReplicaSet OpLog: Enabled                |
➔ |          Commit Hash: db19111d14             |
➔ |        Commit Branch: HEAD                   |
➔ |                                              |
➔ +----------------------------------------------+
```

Now your Rocket.Chat server is fully up and running on your 64bit ARM server!

