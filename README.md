# Mongo DB Sharding

All the configurations here are taken from :

https://medium.com/the-glitcher/mongodb-sharding-9c5357a95ec1

so credits should be given to the author, not me :)

Note that, there was an issue when connecting to the mongo DB, and I had to make a patch, so the docker-compose content here is a bit different than the author.



## Before running up the docker

Execute these commands first to ensure there is no mongod.lock file :

```bash
mkdir -p mongo/data1
mkdir -p mongo/data2
mkdir -p mongo/data3
mkdir -p mongo/data4
mkdir -p mongo/data5
mkdir -p mongo/data6

mkdir -p mongo/config1
mkdir -p mongo/config2
mkdir -p mongo/config3
```

Then run the `docker-compose up -d` command.



After that, run these commands :

##### Configure our config servers replica set

```bash
docker exec -it mongocfg1 bash -c "echo 'rs.initiate({_id: \"mongors1conf\",configsvr: true, members: [{ _id : 0, host : \"mongocfg1\" },{ _id : 1, host : \"mongocfg2\" }, { _id : 2, host : \"mongocfg3\" }]})' | mongo"
```

##### Check config server status

```bash
docker exec -it mongocfg1 bash -c "echo 'rs.status()' | mongo"
```

##### Build shard replica set

```bash
docker exec -it mongors1n1 bash -c "echo 'rs.initiate({_id : \"mongors1\", members: [{ _id : 0, host : \"mongors1n1\" },{ _id : 1, host : \"mongors1n2\" },{ _id : 2, host : \"mongors1n3\" }]})' | mongo"
```

##### Check replica set status

```bash
docker exec -it mongors1n1 bash -c "echo 'rs.status()' | mongo"
```

##### Introduce shard to the routers

```bash
docker exec -it mongos1 bash -c "echo 'sh.addShard(\"mongors1/mongors1n1\")' | mongo "
```

##### Again, check replica set status

```bash
docker exec -it mongors1n1 bash -c "echo 'rs.status()' | mongo"
```

##### Create a database and enable sharding

```bash
docker exec -it mongors1n1 bash -c "echo 'use someDb' | mongo"
```

