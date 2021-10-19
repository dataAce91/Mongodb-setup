# Mongodb-setup

Import the public key used by the package management system
$ wget -qO — https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -

Create the /etc/apt/sources.list.d/mongodb-org-4.2.list
$ echo “deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse” | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list

Reload local package database
$ sudo apt-get update
sudo apt-get install -y mongodb-org
To prevent unintended upgrades
echo “mongodb-org hold” | sudo dpkg — set-selections
echo “mongodb-org-server hold” | sudo dpkg — set-selections
echo “mongodb-org-shell hold” | sudo dpkg — set-selections
echo “mongodb-org-mongos hold” | sudo dpkg — set-selections
echo “mongodb-org-tools hold” | sudo dpkg — set-selections
Start MongoDB
sudo systemctl start mongod
sudo systemctl daemon-reload
To ensure that MongoDB will start following a system reboot
sudo systemctl enable mongod
Verify that MongoDB has started successfully
sudo systemctl status mongod
2. If you have an active DNS server, add A records for all servers, or modify /etc/hosts file. Add these on all nodes.
$ sudo vim /etc/hosts
10.0.0.11 mongoDb-01 
10.0.0.12 mongoDb-02 
10.0.0.13 mongoDb-03
3. Set hostname permanently
$ sudo vim /etc/hostname     ## On Node1 
mongoDb-01 
$ sudo vim /etc/hostname     ## On Node2
mongoDb-02
$ sudo vim /etc/hostname     ## On Node3
mongoDb-03
4. Generate key
On Node 1(mongoDb-01)
# mkdir -p /etc/mongodb/keyFiles/
# openssl rand -base64 756 > /etc/mongodb/keyFiles/mongo-key
# chmod 400 /etc/mongodb/keyFiles/mongo-key
# chown -R mongodb:mongodb /etc/mongodb
copy mongo-key file to all secondary nodes(mongoDb-01,mongoDb-02) at location /etc/mongodb/keyFiles/
Note: create this folder if not present
Step 1: Configure MongoDB Replica set
To configure MongoDB to be ready to run replica sets. The MongoDB configuration can be found at /etc/mongod.conf . Only edit the following parameters. Keep other configurations as default.
On node 1 => mongoDb-01
# network interfaces
net:
  port: 27017
  bindIp: 10.0.0.11
#security:
security:
 authorization: enabled
 keyFile:  /etc/mongodb/keyFile/mongo-key
#replication:
replication:
  replSetName: "replicaset01"
On node 2 => mongoDb-02
# network interfaces
net:
  port: 27017
  bindIp: 10.0.0.12
#security:
security:
 authorization: enabled
 keyFile:  /etc/mongodb/keyFile/mongo-key
#replication:
replication:
  replSetName: "replicaset01"


On node 3=> mongoDb-03
# network interfaces
net:
  port: 27017
  bindIp: 10.0.0.12
#security:
security:
 authorization: enabled
 keyFile:  /etc/mongodb/keyFile/mongo-key
#replication:
replication:
  replSetName: "replicaset01"
To update changes restart mongod service on all nodes
$ sudo systemctl restart mongod
Step 2: Initiate MongoDB Replica Set
Log in to the primary node
On Node 1(mongoDb-01)
# mongo
MongoDB shell version v4.2.1
connecting to: mongodb://10.10.5.2:27017/test
MongoDB server version: 4.0.1
Welcome to the MongoDB shell.
For interactive help, type "help".
...
>
## Initialize replica set on node1 by running below command:
> rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",                                                           
        "me" : "10.10.5.2:27017",
        "ok" : 1,
        "operationTime" : Timestamp(1534797235, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1534797235, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),                                                                          
                        "keyId" : NumberLong(0)
                }
        }
}
OTHER >             # press enter one more time
PRIMARY> 
replicaset01:PRIMARY> 
Step 3: Add secondary nodes
replicaset01:PRIMARY> rs.add("mongoDb-02:27017")
{
        "ok" : 1,
        "operationTime" : Timestamp(1534797580, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1534797580, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}

replicaset01:PRIMARY> rs.add("mongoDb-03:27017")
{
        "ok" : 1,
        "operationTime" : Timestamp(1534797580, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1534797580, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
#To make sure, Display all the replica sets configuration
replicaset01:PRIMARY> rs.status();
Step 4: Set Up a Basic Firewall on all nodes
Ubuntu 18.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall very easily using this application.
# ufw enable
# ufw allow OpenSSH
# ufw allow from <client_ip_address> to any port 27017
To verify
# ufw status
