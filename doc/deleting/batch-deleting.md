---
layout: default
title: Batch Deleting
parent: Deleting
nav_order: 2
---

# Batch Deleting

[Previously](./deleting-a-model.html) we covered deleting a single model by id.
We can also perform more complex, batch deletes using a where clause and
optionally joining tables.

Batch deleting is nearly identical to [Batch
Updating](../updating/batch-updating.html), so you should be familiar with that
section.  Like a batch update, a batch delete starts out with a call to
[DataContext.from](../../api-doc/latest/classes/datacontext.html#from).  That
returns a [FromAdapter](../../api-doc/latest/classes/fromadapter.html) instance,
which can be used to add a where clause, and/or join in additional tables.

* [FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where):
  Add a where clause to the query.
* [FromAdapter.innerJoin](../../api-doc/latest/classes/fromadapter.html#innerjoin):
  Inner join a table.
* [FromAdapter.leftOuterJoin](../../api-doc/latest/classes/fromadapter.html#leftouterjoin):
  Left outer join a table.

The [FromAdapter](../../api-doc/latest/classes/fromadapter.html) class also has
a [delete](../../api-doc/latest/classes/fromadapter.html#delete) method.  It
returns a [Delete](../../api-doc/latest/classes/delete.html) instance that's
used to batch delete records.

Next up, we'll go through some example code that deletes using a where clause
and a join.

### Batch Delete a Person's Phone Numbers

In the [example database](../getting-started/tutorial-database-setup.html), we
defined `people` and `phone_numbers` tables, along with [models](../models/)
for each.  Now let's say we want to delete all the
[PhoneNumber](https://github.com/benbotto/formn-example/blob/1.13.1/src/entity/phone-number.entity.ts)
records associated with a
[Person](https://github.com/benbotto/formn-example/blob/1.13.1/src/entity/person.entity.ts),
and we want to identify the person by `firstName` and `lastName`.  We'll
delete from the `phone_numbers` table, join in `people`, and set an
appropriate where clause.  If you're not familiar with where conditions or
joining, take a look at those sections:
[Joining](../retrieving/joining.html);
[Conditions](../retrieving/conditions.html).

```typescript
// Used for building the delete's where clause.
const cb = new ConditionBuilder();

// Delete all PhoneNumber (pn) records for the Person (p) named
// "Rand Althore."
const query: Delete = dataContext
  .from(PhoneNumber, 'pn')
  .innerJoin(Person, 'p', 'pn.person')
  .where(
    cb.and(
      cb.eq('p.firstName', ':fname', 'Rand'),
      cb.eq('p.lastName', ':lname', 'AlThore')))
  .delete('pn');
```

As mentioned above, a [Delete](../../api-doc/latest/classes/delete.html)
instance is returned.  It's a type of
[Query](../../api-doc/latest/classes/query.html), so we can view the generated
SQL by calling [toString](../../api-doc/latest/classes/delete.html#tostring).

```typescript
console.log(query.toString());
```

To run the query, we
[execute](../../api-doc/latest/classes/delete.html#execute) it.

```typescript
const result: MutateResultType = await query.execute();
```

The `result` will have at least an `affectedRows` property, indicating the
number of deleted rows.

### Full Example

The full code presented above can be found in the formn-example repository
under
[src/delete/delete-phone-numbers-for-person.ts](https://github.com/benbotto/formn-example/blob/1.13.1/src/delete/delete-phone-numbers-for-person.ts).

```typescript
import { inspect } from 'util';

import {
  MySQLDataContext, ConnectionOptions, ConditionBuilder, Delete,
  MutateResultType
} from 'formn';

import { PhoneNumber } from '../entity/phone-number.entity';
import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Used for building the delete's where clause.
    const cb = new ConditionBuilder();

    // Delete all PhoneNumber (pn) records for the Person (p) named
    // "Rand Althore."
    const query: Delete = dataContext
      .from(PhoneNumber, 'pn')
      .innerJoin(Person, 'p', 'pn.person')
      .where(
        cb.and(
          cb.eq('p.firstName', ':fname', 'Rand'),
          cb.eq('p.lastName', ':lname', 'AlThore')))
      .delete('pn');

    console.log(query.toString());

    // The result will have at least an affectedRows property.
    const result: MutateResultType = await query.execute();

    console.log(inspect(result, {depth: null, compact: false}));

    await dataContext.end();
  }
  catch(err) {
    console.error(err);
  }
}

main();
```

Run the example with `ts-node`.

```
npx ts-node ./src/delete/delete-phone-numbers-for-person.ts
```

Output will look like this.

```
DELETE  `pn`
FROM    `phone_numbers` AS `pn`
INNER JOIN `people` AS `p` ON `pn`.`personID` = `p`.`personID`
WHERE   (`p`.`firstName` = :fname AND `p`.`lastName` = :lname)
ResultSetHeader {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 0,
  info: '',
  serverStatus: 34,
  warningStatus: 0
}
```

