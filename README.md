# springboot-vault-consul-demo
This repository is instruction for demo with simple app using Spring Boot + Vault + Consul.

* [UI APP](https://github.com/tkaburagi/springboot-vault-consul-demo-ui)
* [API APP](https://github.com/tkaburagi/springboot-vault-consul-demo-api)

![](https://raw.githubusercontent.com/tkaburagi/springboot-vault-consul-demo/master/diagram.png)


## How to Demo

### Prepartion
* Make sure Java 12 is installed.
* Make sure the latest Consul and Vault is installed.
* Make sure MySQL 5.7 is installed.

First, run Consul and Vault `-dev` mode.
```console
$ export VAULT_ADDR='http://127.0.0.1:8200'
$ vault server -dev
==> Vault server configuration:

             Api Address: http://127.0.0.1:8200
                     Cgo: disabled
         Cluster Address: https://127.0.0.1:8201
              Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: false, enabled: false
                 Storage: inmem
                 Version: Vault v1.1.1
             Version Sha: a3dcd63451cf6da1d04928b601bbe9748d53842e

WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
and starts unsealed with a single unseal key. The root token is already
authenticated to the CLI, so you can immediately begin using Vault.

You may need to set the following environment variable:

    $ export VAULT_ADDR='http://127.0.0.1:8200'

The unseal key and root token are displayed below in case you want to
seal/unseal the Vault or re-authenticate.

Unseal Key: qZAvT/ztCUq8C7liyFG0cG6osIrIi2DtULxAG9zE+nY=
Root Token: s.HEWWO9vQQK2Wc8rYGbKa6n7U

Development mode should NOT be used in production installations!
```
Take `Root Token` value in the note.

```console
$ consul agent -dev
```

Next, run and configure MySQL
```console
$ mysql -uroot
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 469
Server version: 5.7.25 Homebrew

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

```mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
Query OK, 0 rows affected (0.00 sec)

mysql> create database mysqlboot;
Query OK, 1 row affected (0.00 sec)

mysql> use mysqlboot
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> INSERT INTO book
    -> (id, title, author_name, price)
    -> values ("1", "What's HashiCorp", "HashiCorp", "1500");
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO book
    -> (id, title, author_name, price)
    -> values ("2", "eXtream Programming", "Kent Beck", "1200");
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO book
    -> (id, title, author_name, price)
    -> values ("3", "Site Reliability Engineering", "Google", "5600");
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO book
    -> (id, title, author_name, price)
    -> values ("4", "Introduction of Nomad", "Masa Ito", "4900");
Query OK, 1 row affected (0.00 sec)
```

Lastly, configure Vault Database Secret
```console
$ vault secrets enable database

Success! Enabled the database secrets engine at: database/

$ vault write database/config/mysqlboot \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
  allowed_roles="my-role" \
  username="root" \
  password="root"

$ vault write database/roles/my-role \
  db_name=mysqlboot \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

Success! Data written to: database/roles/my-role

$ vault read database/creds/my-role
Key                Value
---                -----
lease_id           database/creds/my-role/3aWbdioHcLQ18rQ0ojWkALwH
lease_duration     1h
lease_renewable    true
password           A1a-UwPd7NKUxkyyxY4d
username           v-root-my-role-FRICYJqHBbxLY55GY
```

### Running API App
```console
$ git clone https://github.com/tkaburagi/springboot-vault-consul-demo-api
```
Add the file, `bootstrap.yml`and edit like below. The value of `token` is the value retreived when Vault started.
```yml
spring:
  application:
    name: book-service
  cloud:
    vault:
      uri: http://127.0.0.1:8200
      token: ((ROOT_TOKEN))
      mysql:
        enabled: true
        role: my-role
        backend: database
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/mysqlboot
```

```console
$ ./mvnw clean package
$ java -jar target/demo-0.0.1-SNAPSHOT.jar --server.port=7070
$ java -jar target/demo-0.0.1-SNAPSHOT.jar --server.port=9090 #Shold be another terminal
```

Make sure API is running and connect to Database via Vault
```console
$ curl http://localhost:8080/allbooks | jq
[
  {
    "id": "1",
    "title": "What's HashiCorp",
    "author_name": "HashiCorp",
    "price": "1500"
  },
  {
    "id": "2",
    "title": "eXtream Programming",
    "author_name": "Kent Beck",
    "price": "1200"
  },
  {
    "id": "3",
    "title": "Site Reliability Engineering",
    "author_name": "Google",
    "price": "5600"
  },
  {
    "id": "4",
    "title": "Introduction of Nomad",
    "author_name": "Masa Ito",
    "price": "4900"
  }
]
```

Let's access to Consul Server(127.0.0.1:8500)
TODO

### Running UI App
```console
$ git clone https://github.com/tkaburagi/springboot-vault-consul-demo-ui
$ ./mvnw clean package
$ java -jar target/demo-0.0.1-SNAPSHOT.jar --server.port=8080
```

Let's access to Consul Server(127.0.0.1:8500) again.
TODO

### Browse UI App
TODO

In the Java terminal of the UI App, you can make sure which API is accessed by UI App
```console

```

### Rotate Database Credential

