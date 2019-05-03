# springboot-vault-consul-demo
This repository is instruction for demo with simple app using Spring Boot + Vault + Consul.

* [UI APP](https://github.com/tkaburagi/springboot-vault-consul-demo-ui)
* [API APP](https://github.com/tkaburagi/springboot-vault-consul-demo-api)

![](https://raw.githubusercontent.com/tkaburagi/springboot-vault-consul-demo/master/diagram.png)


## How to Demo

### Prepartion
* Ensure Java 12 is installed.
* Ensure the latest Consul and Vault is installed.
* Ensure MySQL 5.7 is installed.

First, run Consul and Vault `-dev` mode.
```console
$ export VAULT_ADDR='http://127.0.0.1:8200'
$ vault server -dev
```

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
```