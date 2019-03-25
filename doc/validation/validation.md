---
layout: default
title: Validation
nav_order: 11
has_children: true
permalink: /doc/validation/
---

# Validation

Formn offers some built-in validation classes that can be used prior to
inserting, updating, or deleteing records.  The validation classes build on top
of the [bsy-validation](https://github.com/benbotto/bsy-validation/tree/2.x.x)
package, which provides decorator-based model validation, and is extendable.

Formn models (see the [Models](../models/) section) have a good deal of column
metadata that's provided using the
[Column](../../api-doc/latest/globals.html#column) decorator.  This metadata
includes data type, nullability, max length requirements, and so on.  With that
metadata, Formn can verify that user-provided models meet the criteria needed
to insert, update, or delete records.  For example, if a database table has a
`phoneNumber VARCHAR(255) NOT NULL` column, a corresponding Formn model cannot
be inserted if the `phoneNumber` property is `null`, not a `string`, or exceeds
255 characters.  These metadata verifications, coupled with
[bsy-validation](https://github.com/benbotto/bsy-validation/tree/2.x.x), provide
an extensive validation suite.

