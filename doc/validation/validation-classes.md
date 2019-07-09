---
layout: default
title: Validation Classes
parent: Validation
nav_order: 1
---

# Validation Classes

There are three built-in validation classes in Formn, each of which extends the
[ModelValidator](../../api-doc/latest/classes/modelvalidator.html) base class.

* [InsertModelValidator](../../api-doc/latest/classes/insertmodelvalidator.html):
  Used to validate a model prior to inserting it.
* [UpdateModelValidator](../../api-doc/latest/classes/updatemodelvalidator.html):
  Used to validate a model before updating by ID.
* [DeleteModelValidator](../../api-doc/latest/classes/deletemodelvalidator.html):
  Used to validate a model before deleting by ID.

All three classes work the same way: instantiate the class, then call the
[validate](../../api-doc/latest/classes/modelvalidator.html#validate) method
with two arguments.

1. An object to validate.  This can be a plain JavaScript object, like an
   object from an API request, or an instance of a class.
2. The constructor of a
   [Table](../../api-doc/latest/globals.html#table)-decorated class, a.k.a. an
   "Entity."  The first parameter--the object--will be checked against the
   Entity's metadata to ensure that it's acceptable for the requested CRUD
   operation.

The [validate](../../api-doc/latest/classes/modelvalidator.html#validate)
method returns a
[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
that is resolved if the object is valid, or otherwise rejected with an error
describing why the object is invalid.

In code terms, here's how validation looks.

```typescript
import { InsertModelValidator } from 'formn';

import { PhoneNumber } from '../entity/phone-number.entity';

const validator = new InsertModelValidator();

const phone = {
  personId:    1,
  phoneNumber: '530-222-3333',
  type:        'mobile'
};

// Validate "phone" against the "PhoneNumber" entity, ensuring that "phone" can
// be inserted.
validator
  .validate(phone, PhoneNumber)
  .then(() => console.log('Object is valid.'))
  .catch(err => console.error('Object is invalid: ', err));
```

### Column Metadata

Back in the [Models](../models/) section, we defined a
[PhoneNumber](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts)
class and decorated each property with the
[Column](../../api-doc/latest/globals.html#column) decorator.
[Column](../../api-doc/latest/globals.html#column) takes a
[ColumnMetaOptions](../../api-doc/latest/classes/columnmetaoptions.html) object
as an argument, which provides metadata about each property's corresponding
database column, like the data type, maximum length, nullability, and so on.
Let's look more closely at the `PhoneNumber.phoneNumber` property.

```typescript
@Column({isNullable: false, maxLength: 255, sqlDataType: 'varchar'})
@Validate(new PhoneValidator())
phoneNumber: string;
```

That given, Formn knows that the `phoneNumber` property must be a string, it
must not exceed 255 characters, and it cannot be `null`.  There's also some
custom, user-defined validation provided by the
[bsy-validation](https://github.com/benbotto/bsy-validation/tree/2.x.x)
package.  Namely, we've specified that the `phoneNumber` property should go
through
[bsy-validation](https://github.com/benbotto/bsy-validation/tree/2.x.x)'s
[PhoneValidator](https://github.com/benbotto/bsy-validation/blob/2.x.x/src/validator/phone-validator.ts).

[Next up](validation-rules.html), we'll describe how the validation classes
work in detail, and look over some examples.

