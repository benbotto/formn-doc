---
layout: default
title: Relationships
parent: Models
nav_order: 2
---

# Relationships

In the [last section](./basic-model-decorators.html) we defined a `Person`
class and went over the basic model decorators.  Within a model, properties can
reference other models using one of Formn's relationship decorators,
[OneToMany](../../api-doc/latest/globals.html#onetomany),
[OneToOne](../../api-doc/latest/globals.html#onetoone), or
[ManyToOne](../../api-doc/latest/globals.html#manytoone).

In the example database [we
created](../getting-started/tutorial-database-setup.html#manual-initialization),
there's a one-to-many relationship between `people` and `phone_numbers`.  Let's
create a `PhoneNumber` class, then relate that class to `Person`.

```typescript
import { Table, Column } from 'formn';

@Table({name: 'phone_numbers'})
export class PhoneNumber {
  @Column({name: 'phoneNumberID', isPrimary: true, isGenerated: true,
    isNullable: false, sqlDataType: 'int'})
  id: number;

  @Column({name: 'personID', isNullable: false, sqlDataType: 'int'})
  personId: number;

  @Column({isNullable: false, maxLength: 255, sqlDataType: 'varchar'})
  phoneNumber: string;

  @Column({maxLength: 255, sqlDataType: 'varchar'})
  type: string;
}
```

Each person in our database has one or more phone numbers, so we'll use the
[OneToMany](../../api-doc/latest/globals.html#onetomany) decorator in the
`Person` class.

```typescript
import { Table, Column, OneToMany } from 'formn';

import { PhoneNumber } from './phone-number.entity';

@Table({name: 'people'})
export class Person {
  @Column({name: 'personID', isPrimary: true, isGenerated: true, isNullable: false})
  id: number;

  // ...
  // Other column definitions, as previously defined.
  // ...

  @OneToMany<Person, PhoneNumber>(() => PhoneNumber, (p, pn) => [p.id, pn.personId])
  phoneNumbers: PhoneNumber[];
}
```

All of the relationship
decorators--[OneToMany](../../api-doc/latest/globals.html#onetomany),
[OneToOne](../../api-doc/latest/globals.html#onetoone), and
[ManyToOne](../../api-doc/latest/globals.html#manytoone)--are generic.  They
take two type variables: the type of the local class, `Person` here, and the
type of the related class, in this case `PhoneNumber`.  In other words,
`@OneToMany<Person, PhoneNumber>` is read, "One `Person` has many
`PhoneNumber`s."

Also, all of the relationship decorators have the same method signature (they
take the same arguments).

1. A function that returns the runtime type of the related class.  Above,
   `PhoneNumber` is the related class, so the first parameter is `() =>
   PhoneNumber`.
2. A function that returns an array describing how the two classes are related
   in the underlying database.  Formn uses this information behind the scenes
   to create joins between related tables.  In our example database, a foreign
   key is defined on `phone_numbers.personID`, and it references
   `people.personID`.  Both of those columns are aliased in our model classes,
   `Person.id` and `PhoneNumber.personId`.  So, given `Person` (`p`) and
   `PhoneNumber` (`pn`) instances, these two models are related as `[p.id,
   pn.personId]`.
 
Note that relationships can be composite (i.e. foreign keys can be defined on
multiple columns).  In that case, the second parameter should return an array
of arrays:

```typescript
(left, right) => [[left.prop1, right.relatedProp1], [left.prop2, right.relatedProp2], ...]
```

We can also define the inverse relationship on the `PhoneNumber` class, as each
phone number belongs to a person.

```typescript
import { Table, Column, ManyToOne } from 'formn';

import { Person } from './person.entity';

@Table({name: 'phone_numbers'})
export class PhoneNumber {
  @Column({name: 'personID', isNullable: false})
  personId: number;

  // ...
  // Other column definitions, as previously defined.
  // ...

  @ManyToOne<PhoneNumber, Person>(() => Person, (pn, p) => [pn.personId, p.id])
  person: Person;
}
```
