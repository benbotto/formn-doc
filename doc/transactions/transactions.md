---
layout: default
title: Transactions
nav_order: 10
permalink: /doc/transactions/
---

# Transactions

When you need to batch multiple CRUD operations together it's generally done
in a transaction.  This section covers transactions in Formn.  You should have
a good grasp of the basics ([Getting Started](../getting-started/);
[DataContext](../datacontext/); [Models](../models/)), as well as CRUD
operations ([Creating](../creating/); [Retrieving](../retrieving/);
[Updating](../updating/); [Deleting](../deleting)).

### Starting a Transaction

Transactions are started with a call to the
[DataContext.beginTransaction](../../api-doc/latest/classes/datacontext.html#begintransaction)
method, supplying a function as the single argument.  That user-supplied
function

1. will be called with a new
   [DataContext](../../api-doc/latest/classes/datacontext.html) instance that's
   bound to a single connection, and
2. must return a `Promise`.

```typescript
await dataContext
  .beginTransaction(async (transDataContext: MySQLTransactionalDataContext): Promise<void> => {
    // Apply CRUD operations against transDataContext.
  });
```

Above, an [async
function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
is supplied to
[DataContext.beginTransaction](../../api-doc/latest/classes/datacontext.html#begintransaction),
so an implicit `Promise` is returned.  If the returned `Promise` is rejected
(i.e. if an exception is raised) then the transaction will be automatically
rolled back; otherwise, the transaction will be automatically committed.
Transactions can also be be rolled back manually using the
[DataContext.rollbackTransaction](../../api-doc/latest/classes/mysqltransactionaldatacontext.html#rollbacktransaction)
method.

Importantly, only operations performed against the `transDataContext` will be
part of the transaction; queries run against `dataContext` will not.

### Full Example: Replace PhoneNumbers for a Person

There's an example in the formn-example repository that uses transactions.  It
deletes all the
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
records for a
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts),
and then inserts two new
[PhoneNumbers](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts).
That example can be found under
[src/transaction/replace-phone-numbers.ts](//github.com/benbotto/formn-example/blob/master/src/transaction/replace-phone-numbers.ts),
and it's repeated below for convenience.

```typescript
import { inspect } from 'util';

import {
  MySQLDataContext, ConnectionOptions, ConditionBuilder,
  MySQLTransactionalDataContext, Delete, MutateResultType, Insert
} from 'formn';

import { PhoneNumber } from '../entity/phone-number.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // Start the transaction.
    //
    // Note that the supplied function (replacePhoneNumbers) is async, meaning
    // that it implicitly returns a Promise instance.  If that promise is
    // rejected (i.e. an exception occurs), then the transaction will be rolled
    // back; otherwise, the transaction will be automatically committed.
    await dataContext
      .beginTransaction(replacePhoneNumbers);

    await dataContext.end();
  }
  catch(err) {
    console.error(err);
  }
}

/**
 * Replace the phone numbers for a person.
 */
async function replacePhoneNumbers(
  transDataContext: MySQLTransactionalDataContext): Promise<void> {
  // Delete all phone numbers for the person with ID 1.
  const cb = new ConditionBuilder();

  const qDelPhone: Delete = transDataContext
    .from(PhoneNumber, 'pn')
    .where(cb.eq('pn.personId', ':pid', 1))
    .delete();

  console.log(qDelPhone.toString());

  const qDelPhoneRes: MutateResultType = await qDelPhone.execute();

  console.log(inspect(qDelPhoneRes, {depth: null, compact: false}));

  // Create two new phone numbers for the person with ID 1.
  const phones = [new PhoneNumber(), new PhoneNumber()];

  phones[0].personId    = 1;
  phones[0].phoneNumber = '111-222-3333';
  phones[0].type        = 'cell';

  phones[1].personId    = 1;
  phones[1].phoneNumber = '444-555-6666';
  phones[1].type        = 'office';

  // Insert each phone number, logging the SQL.
  for (let i = 0; i < phones.length; ++i) {
    const qInsPhone: Insert<PhoneNumber> = transDataContext
      .insert(PhoneNumber, phones[i]);

    console.log(qInsPhone.toString());

    await qInsPhone.execute();

    // An id will be set on the PhoneNumber instance.
    console.log(inspect(phones[i], {depth: null, compact: false}));
  }

  // The transaction can be rolled back manually.
  //await transDataContext.rollbackTransaction();
}

main();
```

You can run the example with `ts-node`.

```
npx ts-node ./src/transaction/replace-phone-numbers.ts
```

Output will look similar to the following.

```
DELETE  `pn`
FROM    `phone_numbers` AS `pn`
WHERE   `pn`.`personID` = :pid
ResultSetHeader {
  fieldCount: 0,
  affectedRows: 3,
  insertId: 0,
  info: '',
  serverStatus: 3,
  warningStatus: 0
}
INSERT INTO `phone_numbers` (`personID`, `phoneNumber`, `type`)
VALUES (:personId, :phoneNumber, :type)
PhoneNumber {
  personId: 1,
  phoneNumber: '111-222-3333',
  type: 'cell',
  id: 5
}
INSERT INTO `phone_numbers` (`personID`, `phoneNumber`, `type`)
VALUES (:personId, :phoneNumber, :type)
PhoneNumber {
  personId: 1,
  phoneNumber: '444-555-6666',
  type: 'office',
  id: 6
}
```

