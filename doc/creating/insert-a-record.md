---
layout: default
title: Insert a Record
parent: Creating
nav_order: 1
---

# Insert a Record

Like [connecting](../connecting), query operations in Formn originate with a
[DataContext](../../api-doc/latest/classes/datacontext.html) instance.  The
[DataContext.insert](../../api-doc/latest/classes/datacontext.html#insert)
method is used to insert a model instance into the database.

First, instantiate and populate an instance of a model.  For example, let's add
a
[Person](https://github.com/benbotto/formn-example/blob/1.3.0/src/entity/person.entity.ts).

```typescript
const president = new Person();

president.firstName = 'Abe';
president.lastName  = 'Lincoln';
```

Next, create an [Insert](../../api-doc/latest/classes/insert.html) instance
using the
[DataContext.insert](../../api-doc/latest/classes/datacontext.html#insert)
method.  It takes two parameters.

1. An [EntityType](../../api-doc/latest/globals.html#entitytype), which is a
   [Table](../../api-doc/latest/globals.html#table)-decorated class.  (See the
   [Models](../models) section.)  We're inserting a
   [Person](https://github.com/benbotto/formn-example/blob/1.3.0/src/entity/person.entity.ts).
2. An instance of that [EntityType](../../api-doc/latest/globals.html#entitytype) (`president`).

```typescript
const query: Insert<Person> = dataContext.insert(Person, president);
```

[Insert](../../api-doc/latest/classes/insert.html) _is a_ type of
[Query](../../api-doc/latest/classes/query.html), and, like all Formn queries,
you can call the [toString](../../api-doc/latest/classes/query.html#toString)
method to see the underlying SQL that will Formn will use.

```typescript
console.log(query.toString());
```

Finally, [execute](../../api-doc/latest/classes/query.html#execute) the query
to insert the `president` record.

```typescript
await query.execute();
```

Since
[Person](https://github.com/benbotto/formn-example/blob/1.3.0/src/entity/person.entity.ts)'s
primary key is generated (an auto-incrementing int), Formn sets the generated
`id` on the `president` instance.

### Full Example

Here's the full example, which can be found in the `formn-example` repository
under
[src/create/create-person.ts](https://github.com/benbotto/formn-example/blob/1.3.0/src/create/create-person.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Insert } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Create a new Person instance.
    const president = new Person();

    president.firstName = 'Abe';
    president.lastName  = 'Lincoln';

    // Create an Insert query.
    const query: Insert<Person> = dataContext.insert(Person, president);

    // This is the SQL that will be executed on Query.execute();
    console.log(query.toString());

    // Persist the person.
    await query.execute();

    // The new person has the generated ID set.
    console.log(inspect(president, {depth: null, compact: false}));
  }
  catch(err) {
    console.error('Error creating user.');
    console.error(err);
  }
  finally {
    await dataContext.end();
  }
}

main();
```

Run the example with `ts-node`:

```
npx ts-node ./src/create/create-person.ts
```

The output will look similar to this.

```
INSERT INTO `people` (`firstName`, `lastName`)
VALUES (:firstName, :lastName)
Person {
  firstName: 'Abe',
  lastName: 'Lincoln',
  id: 5
}
```
