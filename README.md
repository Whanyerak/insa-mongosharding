# insa-mongosharding
</br>
# STEPS :
</br>
## Step 1
Lancer le docker-compose.
</br>
## Step 2 : initialiser le Config Server

```
docker exec -it configsvr mongosh --port 27019

rs.initiate({
  _id: "configReplSet",
  configsvr: true,
  members: [{ _id: 0, host: "configsvr:27019" }]
})
```
</br>
## Step 3 : initialiser les Shards

```
docker exec -it shard1 mongosh --port 27018

rs.initiate({
  _id: "shard1ReplSet",
  members: [{ _id: 0, host: "shard1:27018" }]
})

docker exec -it shard2 mongosh --port 27017

rs.initiate({
  _id: "shard2ReplSet",
  members: [{ _id: 0, host: "shard2:27017" }]
})
```
</br>
## Step 4 : ajout des shards au routeur mongos

```
docker exec -it mongos mongosh --port 27020

sh.addShard("shard1ReplSet/shard1:27018")

sh.addShard("shard2ReplSet/shard2:27017")
```
</br>
## Step 5 : ajout d'un champ partagé avec index

```
docker exec -it mongos mongosh --port 27020

use prixdb
db.liste_prix.createIndex({ "COUNTRY": 1 })

sh.enableSharding("prixdb")
sh.shardCollection("prixdb.liste_prix", { "COUNTRY": 1 })
```
</br>
## Step 6 : définir les valeurs de répartition
```
docker exec -it mongos mongosh --port 27020

sh.splitAt("prixdb.liste_prix", { COUNTRY: "FR" })
sh.splitAt("prixdb.liste_prix", { COUNTRY: "GB" })
```
</br>
## Step 7 : déplacer les chunks sur les bons serveurs
```
sh.moveChunk("prixdb.liste_prix", { COUNTRY: "FR" }, "shard1ReplSet")
sh.moveChunk("prixdb.liste_prix", { COUNTRY: "GB" }, "shard2ReplSet")
```
</br>
## Step 8 : créer un utilisateur qui a les droits d'écriture

```
db.createUser({
  user: "admin",
  pwd: "password123",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
```
