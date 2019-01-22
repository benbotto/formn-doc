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
MySQL server.

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

Then run the following SQL.  It creates a `formn-user` user, and initializes
the `people` and `phone_numbers` tables.

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
  ON `formn_test_db`.* to 'formn-user'@'%'
  IDENTIFIED BY 'formn-password';

CREATE TABLE people (
  personID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  firstName VARCHAR(255),
  lastName VARCHAR(255),
  createdOn TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP);

INSERT INTO people (firstName, lastName) VALUES ('Joe', 'Shmo');
INSERT INTO people (firstName, lastName) VALUES ('Rand', 'AlThore');
INSERT INTO people (firstName, lastName) VALUES ('Holly', 'Davis');
INSERT INTO people (firstName, lastName) VALUES ('Jenny', 'Mather');

CREATE TABLE phone_numbers (
  phoneNumberID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  phoneNumber VARCHAR(255) NOT NULL,
  type VARCHAR(255),
  personID INT NOT NULL,

  CONSTRAINT fk__phone_numbers__personID__people
    FOREIGN KEY (personID) REFERENCES people(personID)
    ON DELETE CASCADE);

INSERT INTO phone_numbers (personID, phoneNumber, type) VALUES (1, '530-307-8810', 'mobile');
INSERT INTO phone_numbers (personID, phoneNumber, type) VALUES (1, '916-200-1440', 'home');
INSERT INTO phone_numbers (personID, phoneNumber, type) VALUES (1, '916-293-4667', 'office');
INSERT INTO phone_numbers (personID, phoneNumber, type) VALUES (2, '666-451-4412', 'mobile');
```
