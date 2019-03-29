---
layout: default
title: Validation Rules
parent: Validation
nav_order: 2
---

# Validation Rules

In the [last section](validation-classes.html) we gave an overview of Formn's
built-in validation classes.  Obviously, validation differs between create,
update, and delete operations.  For example, when inserting a
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
instance, the `phoneNumber` property is required: it cannot be `null`; it
cannot be `undefined`.  On the other hand, when updating a
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts),
the `phoneNumber` property can be omitted: the user may want to update
partially, e.g. just change the `type`.

The specific validation rules for each operation are laid out below.  Each has
examples which reference the
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
class, shown here for convenience.

```typescript
import { Validate, PhoneValidator } from 'bsy-validation';

import { Table, Column, ManyToOne } from 'formn';

import { Person } from './person.entity';

@Table({name: 'phone_numbers'})
export class PhoneNumber {
  @Column({name: 'personID', isNullable: false, sqlDataType: 'int'})
  personId: number;

  @Column({isNullable: false, maxLength: 255, sqlDataType: 'varchar'})
  @Validate(new PhoneValidator())
  phoneNumber: string;

  @Column({name: 'phoneNumberID', isPrimary: true, isGenerated: true,
    isNullable: false, sqlDataType: 'int'})
  id: number;

  @Column({maxLength: 255, sqlDataType: 'varchar'})
  type: string;

  @ManyToOne<PhoneNumber, Person>(() => Person, (l, r) => [l.personId, r.id])
  person: Person;
}
```

### InsertModelValidator

Use this class before inserting records.

1. Values associated with generated columns must not be defined.  For example,
   `PhoneNumber.id` cannot be manually set because it's associated with an
   auto-incrementing primary key column.
2. If a column is not nullable and does not have a default value, the associated
   value must be defined.
3. If the object passes steps 1 and 2, the `ModelValidator` rules are applied
   (see below).

The following object is not a valid
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
object because:

1. `id` is associated with a generated column, and thus cannot be defined.
2. `phoneNumber` and `personId` correspond to non-nullable columns, so they
   must be defined.

```typescript
const phone = {
  id: 1
};
```

### UpdateModelValidator

Use this class prior to updating a model by ID.

1. The property or properties associated with the primary key column(s) must be
   defined and cannot be `null`.
2. If the object passes step 1, the `ModelValidator` rules are applied
   (see below).

The below object is not a valid
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
object because:

1. `id` is not defined.

```typescript
const phone = {
  personId: 42,
  type: 'mobile',
  phoneNumber: '530-222-3333'
};
```

### DeleteModelValidator

Use this class prior to deleting a model by ID.

1. The property or properties associated with the primary key column(s) must be
   defined and cannot be `null`.

No other validation is applied.

The following 
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
object cannot be deleted because:

1. `id` is not defined.

```typescript
const phone = {
};
```

Conversely, the following passes validation *even though `phoneNumber` is invalid*, becase:

1. `id` is defined and valid.

```typescript
const phone = {
  id: 1,
  phoneNumber: 'invalid phone number'
};
```

### ModelValidator (The Base Class)

This class is mainly used internally by the above validators.  It could be
useful in certain scenarios, such as making a custom model validator.

1. The data type of each value is validated.
  - For strings, the maximum length is validated.
2. Values corresponding to non-nullable columns must not be `null`.
3. Custom validation rules are applied, such as the `PhoneValidator` check
   described above.

The following
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
object cannot be inserted because:

1. `personId` has an invalid data type (not an integer).
2. `type` has an invalid data type (not a string).

```typescript
const phone = {
  personId: 3.14,
  type: false
};
```

This
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
object cannot be updated because:

1. `phoneNumber` is not a valid phone number.

```typescript
const phone = {
  id: 1,
  phoneNumber: 'bad phone number'
};
```

### Full Example

Each of the example objects presented above is taken from the [validation
directory](http://github.com/benbotto/formn-example/blob/master/src/validation)
of the formn-example repository.  One of the examples,
[src/validation/insert/generated-column-defined.ts](http://github.com/benbotto/formn-example/blob/master/src/validation/insert/generated-column-defined.ts),
is shown below.

```typescript
import { InsertModelValidator } from 'formn';

import { PhoneNumber } from '../../entity/phone-number.entity';

const validator = new InsertModelValidator();

// "id" cannot be defined because it's generated (an auto-incrementing
// primary key).
//
// "phoneNumber" and "personId" are non-nullable, so they must be defined.
const phone = {
  id: 1
};

validator
  .validate(phone, PhoneNumber)
  .catch(err => console.error(JSON.stringify(err, null, 2)));
```

Run it using `ts-node`.

```
npx ts-node src/validation/insert/generated-column-defined.ts
```

Here's the output, which is an `Error` of type
[ValidationErrorList](https://github.com/benbotto/bsy-error/blob/master/src/error/ValidationErrorList.ts).

```
{
  "code": "VAL_ERROR_LIST",
  "name": "ValidationErrorList",
  "detail": "Validation errors occurred.",
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "name": "ValidationError",
      "detail": "\"id\" must not be defined.",
      "field": "id"
    },
    {
      "code": "VALIDATION_ERROR",
      "name": "ValidationError",
      "detail": "\"phoneNumber\" must be defined.",
      "field": "phoneNumber"
    },
    {
      "code": "VALIDATION_ERROR",
      "name": "ValidationError",
      "detail": "\"personId\" must be defined.",
      "field": "personId"
    }
  ]
}
```
