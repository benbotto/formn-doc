---
layout: default
title: Basic Model Decorators
parent: Models
nav_order: 1
---

# Basic Model Decorators

Generally, model classes are generated using the Formn CLI.  We'll cover that
in the [Generating Models](./generating-models.html) section.  For now, we'll
define a model class manually and describe how to make the class Formn capable.

A Formn model is just a TypeScript class decorated with the
[Table](../../api-doc/latest/globals.html#table) decorator.  Within a model
class, properties that correspond to database columns are decorated with the
[Column](../../api-doc/latest/globals.html#column) decorator.  Properties can
also be relational and define relationships with other models, which is covered
in the [next section](./relationships.html).

### Table

Back in the [Getting Started](../getting-started/) section we [defined a
`people`
table](../getting-started/tutorial-database-setup.html#manual-initialization).
Let's make a Formn model for that table.

```typescript
import { Table } from 'formn';

@Table({name: 'people'})
export class Person {
}
```

It's just a plain TypeScipt class, and the
[Table](../../api-doc/latest/globals.html#table) decorator makes it available
for Formn to use.  Note that we've opted to give the class the singular name
`Person`, whereas the underlying database table's name is `people`.  As such,
we need to pass the `name` option to the
[Table](../../api-doc/latest/globals.html#table) decorator.  There are other
[TableMetaOptions](../../api-doc/latest/classes/tablemetaoptions.html), like
`schema` for databases that support schemas.

### Column

Each column in a table can have an associated property in a model class.  Those
properties are [Column](../../api-doc/latest/globals.html#column) decorated.
Like [Table](../../api-doc/latest/globals.html#table), the
[Column](../../api-doc/latest/globals.html#column) decorator can be supplied
various
[ColumnMetaOptions](../../api-doc/latest/classes/columnmetaoptions.html), like
the column name, whether or not the column is nullable, and so on and so forth.

Let's fill out the `Person` model with some
[Column](../../api-doc/latest/globals.html#column)-decorated properties, and
then go over the nitty-gritty.

```typescript
import { Table, Column } from 'formn';

@Table({name: 'people'})
export class Person {
  @Column({name: 'personID', isPrimary: true, isGenerated: true, isNullable: false})
  id: number;

  @Column({maxLength: 255})
  firstName: string;

  @Column({maxLength: 255})
  lastName: string;

  @Column({hasDefault: true, isNullable: false})
  createdOn: Date;
}
```

#### Aliasing Column Names

Database column names and Formn model property names can differ.  Just like we
aliased the `people` table as `Person`, in the above class the
`people.personID` column has been aliased as `id`.  Behind the scenes, Formn
will map `Person.id` to/from `people.personID`.

#### Primary Keys

Every Formn model _must_ have a primary key defined.  A primary key is made up
of one or more [Column](../../api-doc/latest/globals.html#column)-decorated
properties.  In the `people` table, the primary key is defined on the
`personID` column.  As such, the corresponding `id` property has the
`isPrimary` option set in its
[Column](../../api-doc/latest/globals.html#column) decoration.  The `personID`
column is also auto-incrementing (it's generated); hence, the `isGenerated`
option is enabled.

#### Max Length

When we [defined the people
table](../getting-started/tutorial-database-setup.html#manual-initialization),
we defined a maximum character length for the `firstName` and `lastName`
columns.  That restriction is echoed above with the `maxLength` option, which
is useful for validation purposes.

#### Default Values

Database side, `people.createdOn` is defined with a default of
`CURRENT_TIMESTAMP`, and it's not nullable.  For validation purposes, we supply
the `hasDefault` and `isNullable` options to `createdOn`'s
[Column](../../api-doc/latest/globals.html#column) decorator.

