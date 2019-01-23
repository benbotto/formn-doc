---
layout: default
title: Establishing a Connection
parent: Connecting
nav_order: 2
---

# Establishing a Connection

Formn is similar to LINQ to SQL in that most database operations are done via a
[DataContext](../../api-doc/latest/classes/datacontext.html) instance.  There
is one [DataContext](../../api-doc/latest/classes/datacontext.html)
implementation per supported database flavor.  For example, for a MySQL
database you would use a
[MySQLDataContext](../../api-doc/latest/classes/mysqldatacontext.html).  Each
[DataContext](../../api-doc/latest/classes/datacontext.html) implementation
exposes a [connect](../../api-doc/latest/classes/datacontext.html#connect)
method that takes a single
[ConnectionOptions](../../api-doc/latest/classes/connectionoptions.html)
parameter.  Assuming you've followed the tutorial and defined your connection
options in a root-level `connections.json` object (see the [Connection
Options](./connection-options.html) section), code for establishing a
connection will look as follows.

```typescript
import { MySQLDataContext, ConnectionOptions } from 'formn';

async function main() {
  // Load ConnectionOptions from a JSON file.
  const connOpts: ConnectionOptions = require('../../connections.json');

  // Initialize a DataContext instance for MySQL.
  const dataContext = new MySQLDataContext();

  // Attempt to connect.  Log an error on failure, and end the connection on
  // completion.
  try {
    await dataContext.connect(connOpts);
    console.log('Connected!');
  }
  catch(err) {
    console.error('Failed to connect.');
    console.error(err);
  }
  finally {
    await dataContext.end();
  }
}

main();
```

The above example can be found in the formn-example repository under
[`src/connection/connect.ts`](https://github.com/benbotto/formn-example/blob/1.0.0/src/connection/connect.ts).
Run it with ts-node:

```
formn-example$ npx ts-node ./src/connection/connect.ts
```

