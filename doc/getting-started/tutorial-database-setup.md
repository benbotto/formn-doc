---
layout: default
title: Tutorial Database Setup
parent: Getting Started
nav_order: 3
---

# Tutorial Database Setup

All of the example scripts presented in this documentation work against a
fictitious MySQL database that contains people and phone numbers.  If you would
like to follow along, you should create a `formn_test_db` database on a local
MySQL server.  For the sake of simplicity, the tutorial assumes that the
database has a `formn-user` account with full access to the database.  In
production systems, it's best to use a more restricted account (e.g. to limit
the account to CRUD operations).

### Using Docker

The [formn-example](https://github.com/benbotto/formn-example) repository
contains a
[docker-compose.yaml](https://github.com/benbotto/formn-example/blob/master/docker-compose.yml)
manifest file that defines a database container.

```
docker-compose up -d
```

With the container running, a MySQL instance will be accessible on port 3306.
You can access the database for debugging purposes using the MySQL client
directly, or by executing `mysql` in the container.

```
docker-compose exec db mysql -uformn-user -pformn-password -h127.0.0.1 formn_test_db
```

### Manual Initialization

First, create the database.

```sql
CREATE DATABASE formn_test_db
```

Then add a `formn-user` account with full access.

```sql
GRANT ALL PRIVILEGES
  ON `formn_test_db`.* to 'formn-user'@'%'
  IDENTIFIED BY 'formn-password';
```

We'll create some tables and initialize some dummy data in a bit, but first off
let's look at Formn's main interface to the database: the
[DataContext](http://0.0.0.0:4000/doc/formn/5.x.x/api-doc/latest/classes/datacontext.html)
class.
