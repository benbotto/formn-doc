---
layout: default
title: Env Vars
parent: Connecting
nav_order: 3
---

# Env Vars

Formn provides a helper class,
[ConnectionsFileReader](../../api-doc/latest/classes/connectionsfilereader.html),
that can be used to read and validate a `connections.json` file.  Its
[readConnectionOptions](../../api-doc/latest/classes/connectionsfilereader.html#readconnectionoptions)
method takes a path to a `connections.json` file, and returns an array of
[ConnectionOptions](../../api-doc/latest/classes/connectionoptions.html).
Because it's common to use environment variables to store database connection
details, a `connections.json` file can be defined in such a way that some or
all of the connection options come from environment variables.  Here's an
example `connections.json` file that's set up to get the database host and
password from env vars.

```json
{
  "host": {
    "ENV": "DB_HOST"
  },
  "user": "formn-user",
  "password": {
    "ENV": "DB_PASS"
  },
  "database": "formn_test_db",
  "port": 3306,
  "poolSize": 10
}
```

Here, the host is expected to be in the `DB_HOST` environment variable, and the
password in `DB_PASS`.  Once that file is defined, it can be read as follows.

```typescript
import { ConnectionsFileReader, ConnectionOptions } from 'formn';

// Read connection details.  DB_HOST and DB_PASS should be defined
// as env vars, otherwise an exception will be raised.
const connOpts: ConnectionOptions = new ConnectionsFileReader()
  .readConnectionOptions('connections.env.json')[0];
```

There are a few key differences from the [last example](./establishing-a-connection).

1. [readConnectionOptions](../../api-doc/latest/classes/connectionsfilereader.html#readconnectionoptions)
takes the path to a `connections.json` file.  If the path is relative, it's
assumed to be relative to the current working directory, which is likely the
root of your application.  In the [last example](./establishing-a-connection)
the `connections.json` file was simply read using `require`.
2. [readConnectionOptions](../../api-doc/latest/classes/connectionsfilereader.html#readconnectionoptions)
returns an **array** of
[ConnectionOptions](../../api-doc/latest/classes/connectionoptions.html).  Some
applications need to connect to multiple databases, so a `connections.json`
file can define an array of connection details.

The two files referenced above can be found in the formn-example repository:
[connections.env.json](https://github.com/benbotto/formn-example/blob/master/connections.env.json);
[read-connections-env.ts](https://github.com/benbotto/formn-example/blob/master/src/connection/read-connections-env.ts).
