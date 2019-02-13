---
layout: default
title: Custom Join Predicates
parent: Retrieving
nav_order: 6
---

# Custom Join Predicates

In the queries presented in the [last section](./joining.html), the join
predicates (the ON condition) are derived.  That is, Formn infers the join
conditions based on the decorators defined on the models.  But sometimes that's
not enough, and a custom join predicate is needed.  The
[ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html) class
that was covered in the [conditions](./conditions.html) section can be used to
build a custom join condition.

Let's say we want to pull all
[Person](https://github.com/benbotto/formn-example/blob/1.9.0/src/entity/person.entity.ts)
records, along with their mobile
[PhoneNumbers](https://github.com/benbotto/formn-example/blob/1.9.0/src/entity/phone-number.entity.ts),
if any.

```typescript
// The ConditionBuilder is used to build the join predicate.
const cb = new ConditionBuilder();

// People with their mobile phone numbers, if any.
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .leftOuterJoin(PhoneNumber, 'pn', 'p.phoneNumbers',
    cb.and(
      cb.eq('p.id', 'pn.personId'),
      cb.eq('pn.type', ':phoneType', 'mobile')))
  .select();
```

### Full Example

You'll find the above example in the formn-example repository under
[src/retrieve/join-people-to-phone-numbers-custom-condition.ts](https://github.com/benbotto/formn-example/blob/1.9.0/src/retrieve/join-people-to-phone-numbers-custom-condition.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select, ConditionBuilder } from 'formn';

import { Person } from '../entity/person.entity';
import { PhoneNumber} from '../entity/phone-number.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // The ConditionBuilder is used to build the join predicate.
    const cb = new ConditionBuilder();

    // People with their mobile phone numbers, if any.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .leftOuterJoin(PhoneNumber, 'pn', 'p.phoneNumbers',
        cb.and(
          cb.eq('p.id', 'pn.personId'),
          cb.eq('pn.type', ':phoneType', 'mobile')))
      .select();

    console.log(query.toString());

    const people: Person[] = await query
      .execute();

    console.log(inspect(people, {depth: null, compact: false}));

    await dataContext.end();
  }
  catch(err) {
    console.error(err);
  }
}

main();
```

The example can be run with `ts-node`.

```
npx ts-node ./src/retrieve/join-people-to-phone-numbers-custom-condition.ts
```

The above yields the following output.

```
SELECT  `p`.`createdOn` AS `p.createdOn`,
        `p`.`firstName` AS `p.firstName`,
        `p`.`lastName` AS `p.lastName`,
        `p`.`personID` AS `p.id`,
        `pn`.`personID` AS `pn.personId`,
        `pn`.`phoneNumber` AS `pn.phoneNumber`,
        `pn`.`phoneNumberID` AS `pn.id`,
        `pn`.`type` AS `pn.type`
FROM    `people` AS `p`
LEFT OUTER JOIN `phone_numbers` AS `pn` ON (`p`.`personID` = `pn`.`personID` AND `pn`.`type` = :phoneType)
[
  Person {
    id: 1,
    createdOn: 2019-02-13T03:37:23.000Z,
    firstName: 'Joe',
    lastName: 'Shmo',
    phoneNumbers: [
      PhoneNumber {
        id: 1,
        personId: 1,
        phoneNumber: '530-307-8810',
        type: 'mobile'
      }
    ]
  },
  Person {
    id: 2,
    createdOn: 2019-02-13T03:37:23.000Z,
    firstName: 'Rand',
    lastName: 'AlThore',
    phoneNumbers: [
      PhoneNumber {
        id: 4,
        personId: 2,
        phoneNumber: '666-451-4412',
        type: 'mobile'
      }
    ]
  },
  Person {
    id: 3,
    createdOn: 2019-02-13T03:37:23.000Z,
    firstName: 'Holly',
    lastName: 'Davis',
    phoneNumbers: []
  },
  Person {
    id: 4,
    createdOn: 2019-02-13T03:37:23.000Z,
    firstName: 'Jenny',
    lastName: 'Mather',
    phoneNumbers: []
  }
]
```

Note the custom join predicate:

```
LEFT OUTER JOIN `phone_numbers` AS `pn` ON 
  (`p`.`personID` = `pn`.`personID` AND `pn`.`type` = :phoneType)
```

