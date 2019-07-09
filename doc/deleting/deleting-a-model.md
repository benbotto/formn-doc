---
layout: default
title: Deleting a Model
parent: Deleting
nav_order: 1
---

# Deleting a Model

A common scenario is to delete a single model by ID.  That's done using the
[DataContext.delete](../../api-doc/latest/classes/datacontext.html#delete)
method.  The usage is pretty much identical to updating a model by ID, so be
sure to review the [Updating a Model](../updating/updating-a-model.html)
section.  It has more detailed documentation.

## Delete a PhoneNumber Instance

Deleting a record by id is simple.  Below we'll delete a single
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
record.

```typescript
// Delete the PhoneNumber with ID 1.
const phone = new PhoneNumber();

phone.id = 1;

const query: DeleteModel<PhoneNumber> = dataContext
  .delete(PhoneNumber, phone);

console.log(query.toString());

// If no rows are affected (i.e. if a phone number with an id of 1 does not
// exist), then an error will be raised.
await query.execute();
```

First, we instantiate a
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
and set the `id` to 1.  Then we call
[DataContext.delete](../../api-doc/latest/classes/datacontext.html#delete),
supplying two parameters:

1. An [EntityType](../../api-doc/latest/globals.html#entitytype), which is some
   [Table](../../api-doc/latest/globals.html#table)-decorated class.  (See the
   [Models](../models) section.)
2. A model, which is an instance of the
   [EntityType](../../api-doc/latest/globals.html#entitytype).  The model must
   have at least the primary key set.

That returns a [DeleteModel](../../api-doc/latest/classes/deletemodel.html)
instance, which is a type of [Query](../../api-doc/latest/classes/query.html).
Like all Fomrn queries, we can log the SQL that Formn generates using
[DeleteModel.toString](../../api-doc/latest/classes/deletemodel.html#tostring),
and [execute](../../api-doc/latest/classes/deletemodel.html#execute) it.

Note that if there is no record with an `id` of 1, then an error is raised.

### Full Example

The code discussed above is in the formn-example repository under
[src/delete/delete-phone-number.ts](https://github.com/benbotto/formn-example/blob/master/src/delete/delete-phone-number.ts).
It's also printed below for convenience.

```typescript
import {
  MySQLDataContext, ConnectionOptions, DeleteModel
} from 'formn';

import { PhoneNumber } from '../entity/phone-number.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Delete the PhoneNumber with ID 1.
    const phone = new PhoneNumber();

    phone.id = 1;

    const query: DeleteModel<PhoneNumber> = dataContext
      .delete(PhoneNumber, phone);

    console.log(query.toString());

    // If no rows are affected (i.e. if a phone number with an id of 1 does not
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

Run it using `ts-node`.

```
npx ts-node ./src/delete/delete-phone-number.ts
```

Here's the output:

```
DELETE  `phone_numbers`
FROM    `phone_numbers` AS `phone_numbers`
WHERE   (`phone_numbers`.`phoneNumberID` = :phone_numbers_id_0)
```

