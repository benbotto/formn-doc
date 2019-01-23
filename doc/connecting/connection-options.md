---
layout: default
title: Connection Options
parent: Connecting
nav_order: 1
---

# Connection Options

Connection options include a database, host, port, credentials, and a few other
meta-type options.  Take a look at the
[ConnectionOptions](../../api-doc/latest/classes/connectionoptions.html)
documentation for complete details.  By convention, connection options are
usually placed in a `connections.json` file at the root of your project.
Here's an example `connections.json` file.

```json
{
  "host": "127.0.0.1",
  "user": "formn-user",
  "password": "formn-password",
  "database": "formn_test_db",
  "port": 3306,
  "poolSize": 10
}
```

The above JSON object was taken from the formn-example repository.  See
[connections.json](https://github.com/benbotto/formn-example/blob/1.0.0/connections.json).

