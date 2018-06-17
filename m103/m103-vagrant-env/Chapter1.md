

Connect to the vagrant box:
```
cd m103-vagrant-env
vagrant ssh
```
Launch mongod in our first shell:
```
mongod
```
Connect to the Mongo shell in our second shell:
```
mongo
```
Shutdown mongod from our second shell:
```
mongo admin --eval 'db.shutdownServer()'
```
Find command line options for mongod:
```
mongod --help
```
Create new data directory and try to launch mongod with a new port and dbpath, and also fork the process:
```
mkdir first_mongod
mongod --port 30000 --dbpath first_mongod --fork
```
The above command will fail without a logpath - so we add one and then successfully launch mongod:
```
mongod --port 30000 --dbpath first_mongod --logpath first_mongod/mongod.log --fork
```
Try to connect back to Mongo shell, without specifying a port:
```
mongo
```
We need to add a port, because our mongod is running on port 30000, not the default 27017:
```
mongo --port 30000
```
Shutdown the new server:
```
mongo admin --port 30000 --eval 'db.shutdownServer()'
```

## Mongod Configuration

Start mongod from command line:
```
mongod --dbpath /data/db --logpath /data/log/mongod.log --fork --bind_ip "127.0.0.1, 192.168.103.100" --auth --port 27000
```

mongod.conf
```
storage:
  dbPath: "/data/db"
systemLog:
  path: "/data/log/mongod.log"
  destination: "file"
net:
  bindIp : "127.0.0.1,192.168.103.100"
  port : 27000
security:
  authorization: enabled
processManagement:
  fork : true
```
Creating root user:
```
mongo admin --host localhost:27000 --eval '
   db.createUser({
     user: "m103-admin",
     pwd: "m103-pass",
     roles: [
       {role: "root", db: "admin"}
     ]
   })
 '
```

## Basic Commands

User management commands:
```
db.createUser()
db.dropUser()
```
Collection management commands:
```
db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()
```
Database management commands:
```
db.dropDatabase()
db.createCollection()
```
Database status command:
```
db.serverStatus()
```
Creating index with Database Command:
```
db.runCommand(
  { "createIndexes": <collection> },
  { "indexes": [
    {
      "key": { "product": 1 }
    },
    { "name": "name_index" }
    ]
  }
)
```
Creating index with Shell Helper:
```
db.<collection>.createIndex(
  { "product": 1 },
  { "name": "name_index" }
)
```
Introspect a Shell Helper:
```
db.<collection>.createIndex
```

## Logging Basics

Get the logging components:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.getLogComponents()
'
```
Change the logging level:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.setLogLevel(0, "index")
'
```
Tail the log file:
```
tail -f /data/db/mongod.log
```
Update a document:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.products.update( { "sku" : 6902667 }, { $set : { "salePrice" : 39.99} } )
'
```
Look for instructions in the log file with grep:
```
grep -R 'update' /data/db/mongod.log
```

## Profiling the database

Get profiling level:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  use newDB
  db.getProfilingLevel()
'
```
Set profiling level:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.setProfilingLevel(1)
'
```
Show collections:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
show collections
'
```
Set slowms to 0:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.setProfilingLevel( 1, { slowms: 0 } )
'
```
Insert one document into a new collection:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.new_collection.insert( { "a":1 } )
'
```
Get profiling data from system.profile:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.system.profile.find().pretty()
'
```

## Basic Security

Print configuration file:
```
cat /etc/mongod.conf
```
Launch standalone mongod:
```
mongod -f /etc/mongod.conf
```
Connect to mongod:
```
mongo --host 127.0.0.1:27017
```
Create new user with the root role (also, named root):
```
use admin
db.createUser({
  user: "root",
  pwd: "root123",
  roles : [ "root" ]
})
```
Connect to mongod and authenticate as root:
```
mongo --username root --password root123 --authenticationDatabase admin
```
Run DB stats:
```
db.stats()
```
Shutdown the server:
```
use admin
db.shutdownServer()
```

## Built-in roles

Authenticate as root user:
```
mongo admin -u root -p root123
```
Create security officer:
```
db.createUser(
  { user: "security_officer",
    pwd: "h3ll0th3r3",
    roles: [ { db: "admin", role: "userAdmin" } ]
  }
)
```
Create database administrator:
```
db.createUser(
  { user: "dba",
    pwd: "c1lynd3rs",
    roles: [ { db: "admin", role: "dbAdmin" } ]
  }
)
```
Grant role to user:
```
db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )
```
Show role privileges:
```
db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} )
```

## Server Tools Overview

List mongodb binaries:
```
find /usr/bin/ -name "mongo*"
```
Create new dbpath and launch mongod:
```
mkdir -p ~/first_mongod
mongod --port 30000 --dbpath ~/first_mongod --logpath ~/first_mongod/mongodb.log --fork
```
Use mongostat to get stats on a running mongod process:
```
mongostat --help
mongostat --port 30000
```
Use mongodump to get a BSON dump of a MongoDB collection:
```
mongodump --help
mongodump --port 30000 --db applicationData --collection products
ls dump/applicationData/
cat dump/applicationData/products.metadata.json
```
Use mongorestore to restore a MongoDB collection from a BSON dump:
```
mongorestore --drop --port 30000 dump/
```
Use mongoexport to export a MongoDB collection to JSON or CSV (or stdout!):
```
mongoexport --help
mongoexport --port 30000 --db applicationData --collection products
mongoexport --port 30000 --db applicationData --collection products -o products.json
```
Tail the exported JSON file:
```
tail products.json
```
Use mongoimport to create a MongoDB collection from a JSON or CSV file:
```
mongoimport --port 30000 products.json
mongoimport --host 192.168.103.100 --port 27000 --username m103-admin --password m103-pass --authenticationDatabase admin  /dat
aset/products.json --db applicationData --collection products
```

