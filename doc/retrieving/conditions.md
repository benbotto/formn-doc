---
layout: default
title: Conditions
parent: Retrieving
nav_order: 4
---

# Conditions

Formn uses a domain-specific language for WHERE and ON conditions, and
conditions are lexed, parsed, and compiled.  This approach is pretty flexible,
as it lets developers create conditions on the fly, or even accept condition
filters via an API feed.

Condition expressions are given in prefix notation rather than infix, so if
you've used MongoDB the conditions will look familiar.  Let's take a look at
some examples.

### Basic Comparisons

The [FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where)
method is used to filter a query.  It takes two parameters: a condition object,
and (optionally) any query parameters used in the condition.

A basic comparison takes the form:

```
{<comparison-operator>: {<model-property> : <value>}}
```

The basic comparison operators are `$eq`, `$neq`, `$lt`, `$lte`, `$gt`, `$gte`,
`$like`, and `$notlike`.  For example, if we wanted to find all 
[Person](https://github.com/benbotto/formn-example/blob/1.7.0/src/entity/person.entity.ts)
records with a `firstName` of "Joe," we would use the `$eq` operator.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    {$eq: {'p.firstName' : ':myFirstName'}},
    {myFirstName: 'Joe'})
  .select();

// Resulting condition: WHERE   `p`.`firstName` = :myFirstName
```

Or, if we wanted to find all people created after New Years, 2019.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    {$gt: {'p.createdOn' : ':newyears'}},
    {newyears: new Date('2019-01-01')})
  .select();

// Resulting condition: WHERE   `p`.`createdOn` > :newyears
```

### Logical (Aggregate) Conditions

Conditions can be combined using the `$and` and `$or` logical operators.  These
types of conditions take the form:

```
{<logical-operator>: [<condition>, ...]}
```

That is, an operator proceeded by an array of conditions.

So let's say we want to find all
[Person](https://github.com/benbotto/formn-example/blob/1.7.0/src/entity/person.entity.ts)
records that have a "j" in their first or last name.

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    {
      $or: [
        {$like: {'p.firstName': ':letter'}},
        {$like: {'p.lastName': ':letter'}}
      ]
    },
    {letter: '%j%'})
  .select();

// Resulting condition: WHERE   (`p`.`firstName` LIKE :letter OR `p`.`lastName` LIKE :letter)
```

Above, two `$like` comparisons are `OR`'d together.  Now let's find people with
a "j" and an "r" in the name (e.g. "Jenny Mather").

```typescript
const query: Select<Person> = dataContext
  .from(Person, 'p')
  .where(
    {
      $and: [
        {
          $or: [
            {$like: {'p.firstName': ':letter1'}},
            {$like: {'p.lastName': ':letter1'}}
          ]
        },
        {
          $or: [
            {$like: {'p.firstName': ':letter2'}},
            {$like: {'p.lastName': ':letter2'}}
          ]
        }
      ]
    },
    {letter1: '%j%', letter2: '%r'})
  .select();

// Resulting condition:
// WHERE   ((`p`.`firstName` LIKE :letter1 OR `p`.`lastName` LIKE :letter1) AND
//          (`p`.`firstName` LIKE :letter2 OR `p`.`lastName` LIKE :letter2))
```

Obviously, Formn places parentheses around logically-grouped conditions so
that the operator precedence is correct.

### Condition Language Spec

As mentioned above, Formn conditions follow a formal spec.  The BNF can be
found
[here](https://github.com/benbotto/formn/blob/5.0.0/src/query/condition/condition-bnf.txt).

### Full Example

You can see the above example in the formn-example repository under
[src/retrieve/conditions.ts](https://github.com/benbotto/formn-example/blob/1.7.0/src/retrieve/conditions.ts).

```typescript
import { inspect } from 'util';

import { MySQLDataContext, ConnectionOptions, Select } from 'formn';

import { Person } from '../entity/person.entity';

async function main() {
  const connOpts: ConnectionOptions = require('../../connections.json');
  const dataContext = new MySQLDataContext();

  try {
    await dataContext.connect(connOpts);

    // People with a "j" and an "r" in their name, like Jenny Mather.
    const query: Select<Person> = dataContext
      .from(Person, 'p')
      .where(
        {
          $and: [
            {
              $or: [
                {$like: {'p.firstName': ':letter1'}},
                {$like: {'p.lastName': ':letter1'}}
              ]
            },
            {
              $or: [
                {$like: {'p.firstName': ':letter2'}},
                {$like: {'p.lastName': ':letter2'}}
              ]
            }
          ]
        },
        {letter1: '%j%', letter2: '%r'})
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

