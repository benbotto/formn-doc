---
layout: default
title: Batch Updating
parent: Updating
nav_order: 2
---

# Batch Updating

In the [previous section](./updating-a-model.html) we updated a single model by
ID.  While that's a common scenario, often there's a need to update a bunch of
records simultaneously based on some filter condition.

If you look back at the [DataContext](../datacontext) section, you'll see that
the [DataContext.from](../../api-doc/latest/classes/datacontext.html#from)
method can be used for batch updating.  As was mentioned in the [Basic
Retrieval](../retrieving/basic-retrieval.html) section, the
[DataContext.from](../../api-doc/latest/classes/datacontext.html#from) method
returns a [FromAdapter](../../api-doc/latest/classes/fromadapter.html)
instance.  That class has an
[update](../../api-doc/latest/classes/fromadapter.html#update) method that can
be used to perform batch updates.

### Update With a Where Clause

In the last chapter, [Conditions](../retrieving/conditions.html) were covered.
Because both retrieving and batch updating share a common inteface--namely
[FromAdapter](../../api-doc/latest/classes/fromadapter.html)--both are filtered
in the same way: using the
[FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where) method
along with the
[ConditionBuilder](../../api-doc/latest/classes/conditionbuilder.html) class.
(You can also use join operations, as described in
[Joining](../retrieving/joining.html) and [Batch
Deleting](../deleting/batch-deleting.html).)

When we created the [example
database](../getting-started/tutorial-database-setup.html), we added `type`
column to the `phone_numbers` table.  Let's change all "mobile" phone numbers
to "cell."

```typescript
// Used for building the update statement's where clause.
const cb = new ConditionBuilder();

// Change phone_numbers.type from "mobile" to "cell."
const query: Update = dataContext
  .from(PhoneNumber, 'pn')
  .where(cb.eq('pn.type', ':mob', 'mobile'))
  .update({'pn.type': 'cell'});
```

In the above code, we've used the
[DataContext.from](../../api-doc/latest/classes/datacontext.html#from) method,
supplying two parameters:

1. An [EntityType](../../api-doc/latest/globals.html#entitytype), which is some
   [Table](../../api-doc/latest/globals.html#table)-decorated class.  (See the
   [Models](../models) section.)
2. An alias, which lets us reference the
   [EntityType](../../api-doc/latest/globals.html#entitytype) using a
   short-hand notation (`pn` above).

We then used the
[FromAdapter.where](../../api-doc/latest/classes/fromadapter.html#where) method
to filter the query, limiting the update to mobile-type phone numbers.
(Building conditions and filtering queries is covered in greater depth in the
[Conditions](../retrieving/conditions.html) section.)  Lastly, we called
[FromAdapter.update](../../api-doc/latest/classes/fromadapter.html#update),
supplying a single object as a parameter.  Each key in the object corresponds
to a fully-qualified property name in the form `<table-alias>.<property-name>`.
Above, we set `pn.type` to "cell."

[FromAdapter.update](../../api-doc/latest/classes/fromadapter.html#update)
returns an [Update](../../api-doc/latest/classes/update.html) instance.  And
like all Formn queries, that class inherits from
[Query](../../api-doc/latest/classes/query.html).  As usual, we can log the SQL
for debugging purposes using
[Update.toString](../../api-doc/latest/classes/update.html#tostring).

```typescript
console.log(query.toString());
```

And to run the query, we call
[Update.execute](../../api-doc/latest/classes/update.html#execute).

```typescript
const result: MutateResultType = await query.execute();
```

The returned
[MutateResultType](../../api-doc/latest/globals.html#mutateresulttype) will
have at least an `affectedRows` property describing how many rows were updated.

### Full Example

The example presented above is located under
[src/update/batch-update-mobile-numbers.ts](https://github.com/benbotto/formn-example/blob/master/src/update/batch-update-mobile-numbers.ts)
in the formn-example repository.  Run it with `ts-node`.

```
npx ts-node ./src/update/batch-update-mobile-numbers.ts
```

The output is shown below.

```
UPDATE  `phone_numbers` AS `pn`
SET
`pn`.`type` = :pn_type_0
WHERE   `pn`.`type` = :mob
ResultSetHeader {
  fieldCount: 0,
  affectedRows: 2,
  insertId: 0,
  info: 'Rows matched: 2  Changed: 2  Warnings: 0',
  serverStatus: 34,
  warningStatus: 0,
  changedRows: 2
}
```

Note that only the `affectedRows` property is guaranteed, but additional
metadata may be present depending on the underlying database driver.

