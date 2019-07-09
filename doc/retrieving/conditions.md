---
layout: default
title: Conditions
parent: Retrieving
nav_order: 4
---

# Conditions

The easiest way to create conditions with Formn is to use the
[ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html) class.
You build a condition, and then pass it to the
[FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where) method
to filter a query.

### Basic Comparisons

The [ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html) class
provides these basic comparison functions.

* [eq](../../api-doc/latest/classes/conditionbuilder.html#eq): Equal (`=`)
  comparison.
* [neq](../../api-doc/latest/classes/conditionbuilder.html#neq): Not equal
  (`<>`) comparison.
* [lt](../../api-doc/latest/classes/conditionbuilder.html#lt): Less than (`<`)
  comparison.
* [lte](../../api-doc/latest/classes/conditionbuilder.html#lte): Less than or
  equal to (`<=`) comparison.
* [gt](../../api-doc/latest/classes/conditionbuilder.html#gt): Greater than
  (`>`) comparison.
* [gte](../../api-doc/latest/classes/conditionbuilder.html#gte): Greater than
  or equal to (`>=`) comparison.
* [like](../../api-doc/latest/classes/conditionbuilder.html#like): Like
  (`LIKE`) comparison.
* [notLike](../../api-doc/latest/classes/conditionbuilder.html#notlike): Not
  like (`NOT LIKE`) comparison.
* [in](../../api-doc/latest/classes/conditionbuilder.html#in): In (`IN (...)`)
  comparison.
* [notIn](../../api-doc/latest/classes/conditionbuilder.html#notin): Not in
  (`NOT IN (...)`) comparison.
* [isNull](../../api-doc/latest/classes/conditionbuilder.html#isnull): Is null
  (`IS NULL`) comparison.
* [isNotNull](../../api-doc/latest/classes/conditionbuilder.html#isnotnull): Is
  not null (`IS NOT NULL`) comparison.

In a nutshell, you build a condition, then pass the result to the
[FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where)
method.  For example, if we wanted to find all
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records with a `firstName` of "Joe," we would use the
[eq](../../api-doc/latest/classes/conditionbuilder.html#eq) method.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    cb.eq('p.firstName', ':myFirstName', 'Joe'))
  .select();

// Resulting condition: WHERE   `p`.`firstName` = :myFirstName
```

Or, if we wanted to find all people created after New Year's, 2019.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    cb.gt('p.createdOn', ':newyears', new Date('2019-01-01')))
  .select();

// Resulting condition: WHERE   `p`.`createdOn` > :newyears
```

### Logical (Aggregate) Conditions

Conditions can be combined using the
[and](../../api-doc/latest/classes/conditionbuilder.html#and) and
[or](../../api-doc/latest/classes/conditionbuilder.html#or) methods.  Let's say
we want to find all
[Person](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
records that have a "j" in the first or last name.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    cb.or(
      cb.like('p.firstName', ':letter', '%j%'),
      cb.like('p.lastName', ':letter', '%j%')))
  .select();

// Resulting condition: WHERE   (`p`.`firstName` LIKE :letter OR `p`.`lastName` LIKE :letter)
```

Above, two basic `LIKE` comparisons are `OR`'d together.  Now let's find people
with a "j" and an "r" in the first or last name (e.g. "Jenny Mather").

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    cb.and(
      cb.or(
        cb.like('p.firstName', ':letter1', '%j%'),
        cb.like('p.lastName', ':letter1', '%j%')),
      cb.or(
        cb.like('p.firstName', ':letter2', '%r%'),
        cb.like('p.lastName', ':letter2', '%r%'))))
  .select();

// Resulting condition:
// WHERE   ((`p`.`firstName` LIKE :letter1 OR `p`.`lastName` LIKE :letter1) AND
//          (`p`.`firstName` LIKE :letter2 OR `p`.`lastName` LIKE :letter2))
```

Obviously, Formn places parentheses around logically-grouped conditions so
that the operator precedence is correct.

### Conditions from Objects (Advanced)

The [FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where)
method is overloaded.  Rather than using the
[ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html) class,
you can pass two parameters:

* A condition object.
* A parameter object that contains key-value pairs.  Each key corresponds to a
  parameter in the condition object.

The [ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html)
class just builds condition objects using a simple interface.  After building a
condition, you can see the resulting condition object using the
[ParameterizedCondition.getCond](../../api-doc/latest/classes/parameterizedcondition.html#getcond)
method.

```typescript
const pCond = cb.and(
  cb.or(
    cb.like('p.firstName', ':letter1', '%j%'),
    cb.like('p.lastName', ':letter1', '%j%')),
  cb.or(
    cb.like('p.firstName', ':letter2', '%r%'),
    cb.like('p.lastName', ':letter2', '%r%')));

console.log(inspect(pCond.getCond(), {depth: null, compact: false}));

/*
{
  '$and': [
    {
      '$or': [
        {
          '$like': {
            'p.firstName': ':letter1'
          }
        },
        {
          '$like': {
            'p.lastName': ':letter1'
          }
        }
      ]
    },
    {
      '$or': [
        {
          '$like': {
            'p.firstName': ':letter2'
          }
        },
        {
          '$like': {
            'p.lastName': ':letter2'
          }
        }
      ]
    }
  ]
}
*/
```

The objects are in prefix notation, similar to MongoDB's conditions, and they
follow a formal language specification.  That spec can be found
[here](https://github.com/benbotto/formn/blob/5.0.0/src/query/condition/condition-bnf.txt).

Using condition objects, one could, for example, accept dynamic conditions via
an API, or build complex conditions on the fly.

### Full Example

You can see the above example in the formn-example repository under
[src/retrieve/conditions.ts](https://github.com/benbotto/formn-example/blob/master/src/retrieve/conditions.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select, ConditionBuilder } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // The ConditionBuilder class is used to build WHERE or ON conditions.
    const cb = new ConditionBuilder();

    // People with a "j" and an "r" in their name, like Jenny Mather.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .where(
        cb.and(
          cb.or(
            cb.like('p.firstName', ':letter1', '%j%'),
            cb.like('p.lastName', ':letter1', '%j%')),
          cb.or(
            cb.like('p.firstName', ':letter2', '%r%'),
            cb.like('p.lastName', ':letter2', '%r%'))))
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

It can be run with `ts-node`:

```
npx ts-node ./src/retrieve/conditions.ts
```

The output will look something like this:

```
SELECT  `p`.`createdOn` AS `p.createdOn`,
        `p`.`firstName` AS `p.firstName`,
        `p`.`lastName` AS `p.lastName`,
        `p`.`personID` AS `p.id`
FROM    `people` AS `p`
WHERE   ((`p`.`firstName` LIKE :letter1 OR `p`.`lastName` LIKE :letter1) AND (`p`.`firstName` LIKE :letter2 OR `p`.`lastName` LIKE :letter2))
[
  Person {
    id: 4,
    createdOn: 2019-01-24T07:14:07.000Z,
    firstName: 'Jenny',
    lastName: 'Mather'
  }
]
```

