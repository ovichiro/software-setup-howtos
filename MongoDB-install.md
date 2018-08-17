MongoDB quick install guide
===============================

This setup will focus on the Ubuntu Linux distro.

The installation steps are detailed on the official site, at https://docs.mongodb.com/manual/administration/install-on-linux/


### Install

```sh
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

$ echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list

$ sudo apt-get update

$ sudo apt-get install -y mongodb-org
```


### Run

Start service: `$ sudo service mongod start`  
Check status: `$ sudo service mongod status`  
Stop service: `$ sudo service mongod stop`  

*Note: On Windows it should be started with the auth param - `$ mongod.exe --auth`*


### Configure networking access

Depending on where you install MongoDB - container, VM, local - you may need to change the `bind_ip` property in order for the instance to be accessible from your dev machine.  

Example values: `bind_ip: 127.0.0.1`, `bind_ip: [127.0.0.1,192.168.100.101]`  

Edit /etc/mongod.conf

```
net:
    bindIp: 127.0.0.1
    port: 27017
```

Make sure that MongoDB is only accessible from the local network, not the Internet.


### Configure authentication and users

The DB offers unrestricted access initially.  
First create the needed users, then enable authentication and restart the `mongod` service.  

Run the Mongo shell from the terminal: 

```
$ mongo
```

Create *admin* and *regular* users in the Mongo shell:

```js
> use admin

> db.createUser(
  {
    user: "dbadmin",
    pwd: "AdminPass123",
    roles: [{role: "userAdminAnyDatabase", db: "admin"}]
  }
)

> db.createUser(
  {
    user: "dbroot",
    pwd: "RootPass123",
    roles: ["root"]
  }
)

> db.createUser(
  {
    user: "user",
    pwd: "UserPass123",
    roles: [
        {role: "dbAdmin", db: "mydb"},
        {role: "readWrite", db: "mydb"}
    ]
  }
)
```

Enable authentication by editing /etc/mongod.conf

```
security:
    authorization: enabled
```

Restart the Mongo service:

```
$ service mongod restart
```

Example login with authenticated user

```
$ mongo

> use admin
> db.auth("user", "UserPass123")
```

Details on authentication: https://docs.mongodb.com/manual/tutorial/enable-authentication/  
Details on roles: https://docs.mongodb.com/manual/core/security-built-in-roles/#all-database-roles


### Create app database

To create a database in MongoDB, we just need to define a name and insert a document in it.

From the Mongo shell:

```
> use mydb
> db.mycollection.insert({name: "First insert to create the db and collection."});
```

The above commands translate to:

 - switch to db named `mydb` or define it if not existing
 - create `mydb` database
 - create `mycollection` collection (the equivalent of a table in a relational DB)
 - insert the document defined by the JSON `{name: "First insert to create the db."}`

For a quick tutorial on CRUD syntax, check out http://www.newthinktank.com/2015/12/mongodb-tutorial-2/


### Tools

Robomongo: https://robomongo.org/

