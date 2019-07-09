---
layout: default
title: Joining
parent: Retrieving
nav_order: 5
---

# Joining

Generally, joining two tables is straight forward.  How two models are related
is defined on the models using relationship decorators,
[OneToMany](../../api-doc/latest/globals.html#onetomany),
[OneToOne](../../api-doc/latest/globals.html#onetoone), or
[ManyToOne](../../api-doc/latest/globals.html#manytoone).  Formn uses that
metadata to join the tables together.

For example, back when we [defined our
models](../models/generating-models.html), we specified a one-to-many
relationship between
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
and
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts),
and a many-to-one relationship in the opposite direction.  With those
relationships in place, we can pull all the
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records along with each
[Person's](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
[PhoneNumbers](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts).
We do that with one of the
[FromAdapter's](../../api-doc/latest/classes/fromadapter.html) join methods:
[leftOuterJoin](../../api-doc/latest/classes/fromadapter.html#leftouterjoin) or
[innerJoin](../../api-doc/latest/classes/fromadapter.html#innerjoin).  Here's an example:

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .leftOuterJoin(PhoneNumber, 'pn', 'p.phoneNumbers')
  .select();
```

That says to join from `people` to `phone_numbers`, using the relationship
defined on the `p.phoneNumbers` property.  You'll get back all
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records, and each
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
will have an array of
[PhoneNumbers](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts).

If you wanted to go the other direction, joining from `phone_numbers` to
`people`:

```typescript
const query: Select<PhoneNumber> = dataContext
  .from(PhoneNumber, 'pn')
  .innerJoin(Person, 'p', 'pn.person')
  .select();
```

which would give you all
[PhoneNumbers](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts),
each with a
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
instance.

### Full Example

Both examples presented above are available in the formn-example repository.  Take a look at
[src/retrieve/join-people-to-phone-numbers.ts](https://github.com/benbotto/formn-example/blob/master/src/retrieve/join-people-to-phone-numbers.ts)
and
[src/retrieve/join-phone-numbers-to-people.ts](https://github.com/benbotto/formn-example/blob/master/src/retrieve/join-phone-numbers-to-people.ts).
The former is presented below.

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select } from 'formn';

import { Person } from '../entity/person.entity';
import { PhoneNumber} from '../entity/phone-number.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // People with their phone numbers.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .leftOuterJoin(PhoneNumber, 'pn', 'p.phoneNumbers')
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

As usual, run it with `ts-node`.

```
npx ts-node ./src/retrieve/join-people-to-phone-numbers.ts
```

The output will look similar to the following.

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
LEFT OUTER JOIN `phone_numbers` AS `pn` ON `p`.`personID` = `pn`.`personID`
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
      },
      PhoneNumber {
        id: 2,
        personId: 1,
        phoneNumber: '916-200-1440',
        type: 'home'
      },
      PhoneNumber {
        id: 3,
        personId: 1,
        phoneNumber: '916-293-4667',
        type: 'office'
      }
    ]
  },
  Person {
    id: 2,
    createdOn: 2019-02-13T03:37:23.000Z,
    firstName: 'Rand',
    lastName: 'al\'Thor',
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

