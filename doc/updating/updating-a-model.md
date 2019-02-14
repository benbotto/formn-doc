---
layout: default
title: Updating a Model
parent: Updating
nav_order: 1
---

# Updating a Model

A [model](../models/) is updating using the
[DataContext.update](../../api-doc/latest/classes/datacontext.html#update)
method.  It taks two parameters:

1. An [EntityType](../../api-doc/latest/globals.html#entitytype), which is some
   [Table](../../api-doc/latest/globals.html#table)-decorated class.  (See the
   [Models](../models) section.)
2. A model, which is an instance of the
   [EntityType](../../api-doc/latest/globals.html#entitytype).  The model must
   have at least the primary key set.

[DataContext.update](../../api-doc/latest/classes/datacontext.html#update)
returns an [UpdateModel](../../api-doc/latest/classes/updatemodel.html)
instance, which is a type of [Query](../../api-doc/latest/classes/query.html).
As such, you can use the
[toString](../../api-doc/latest/classes/updatemodel.html#tostring) method for
debugging, and the
[execute](../../api-doc/latest/classes/updatemodel.html#execute) method to run
the query.

### Update a Person Instance

Here's how one could update the `firstName` of a
[Person](https://github.com/benbotto/formn-example/blob/1.10.1/src/entity/person.entity.ts)
record.

```typescript
// Update the first name of the person with ID 1.
const person = new Person();

person.id = 1;
person.firstName = 'Marco';

const query: UpdateModel<Person> = dataContext
  .update(Person, person);

console.log(query.toString());

// If no rows are affected (i.e. if a person with an id of 1 does not
// exist), then an error will be raised.
await query.execute();
```

In this case, we've only updated one property, `firstName`; all other
properties are ignored.  It's worth pointing out that Formn differentiates
between `undefined` and `null`.  In the above query, since `person.lastName` is
`undefined`, Formn ignores it.  If it were `null`, however, Formn would set the
associated value to `NULL` in the database.

Also note the final comment.  Updating a model is done by ID, and if the update
does not affect any rows then an error will be raised.  In the [next
section](./batch-updating.html) we'll cover batch updates, which do not have
this restriction.

### Full Example

The example shown above is present in the formn-example repository under
[src/update/update-person.ts](https://github.com/benbotto/formn-example/blob/1.10.1/src/update/update-person.ts).

```typescript
import {
  MySQLDataContext, ConnectionOptions, ConditionBuilder, UpdateModel
} from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Update the first name of the person with ID 1.
    const person = new Person();

    person.id = 1;
    person.firstName = 'Marco';

    const query: UpdateModel<Person> = dataContext
      .update(Person, person);

    console.log(query.toString());

    // If no rows are affected (i.e. if a person with an id of 1 does not
    // exist), then an error will be raised.
    await query.execute();
    await dataContext.end();
  }
  catch(err) {
    console.error(err);
  }
}

main();
```

It can be run with `ts-node`.

```
npx ts-node ./src/update/update-person.ts
```

Output will look like this.

```
UPDATE  `people` AS `people`
SET
`people`.`firstName` = :people_firstName_0
WHERE   (`people`.`personID` = :people_id_0)
```

Again, note that only the `firstName` is updated, and the update is by ID.

