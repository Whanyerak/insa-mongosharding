# insa-mongosharding

## STEPS :

### Step 1
Lancer le docker-compose.
</br>
</br>

### Step 2 : initialiser le Config Server

```
docker exec -it configsvr mongosh --port 27019

rs.initiate({
  _id: "configRS",
  configsvr: true,
  members: [{ _id: 0, host: "configsvr:27019" }]
})
```

### Step 3 : initialiser les Shards

```
docker exec -it shard1 mongosh --port 27018

rs.initiate({
  _id: "shard1RS",
  members: [{ _id: 0, host: "shard1:27018" }]
})

docker exec -it shard2 mongosh --port 27017

rs.initiate({
  _id: "shard2RS",
  members: [{ _id: 0, host: "shard2:27017" }]
})
```

### Step 4 : ajout des shards au routeur mongos

```
docker exec -it mongos mongosh --port 27020

sh.addShard("shard1RS/shard1:27018")

sh.addShard("shard2RS/shard2:27017")
```

### Step 5 : ajout d'un champ partagé avec index

```
docker exec -it mongos mongosh --port 27020

use pricesdb
db.prices.createIndex({ "COUNTRY": 1 })

sh.enableSharding("pricesdb")
sh.shardCollection("pricesdb.prices", { "COUNTRY": 1 })
```

### Step 6 : définir les valeurs de répartition
```
docker exec -it mongos mongosh --port 27020

sh.splitAt("pricesdb.prices", { COUNTRY: "FR" })
sh.splitAt("pricesdb.prices", { COUNTRY: "GB" })
```

## Step 7 : déplacer les chunks sur les bons serveurs
```
sh.moveChunk("pricesdb.prices", { COUNTRY: "FR" }, "shard1RS")
sh.moveChunk("pricesdb.prices", { COUNTRY: "GB" }, "shard2RS")
```

## Step 8 : créer un utilisateur qui a les droits d'écriture

```
docker exec -it mongos mongosh --port 27020

use admin
db.createUser({
  user: "admin",
  pwd: "password123",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
```
