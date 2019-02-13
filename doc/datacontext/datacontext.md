---
layout: default
title: DataContext
nav_order: 3
permalink: /doc/datacontext/
---

# DataContext

Formn is similar to LINQ to SQL in that most database operations are done via a
[DataContext](../../api-doc/latest/classes/datacontext.html) instance.  There
is one [DataContext](../../api-doc/latest/classes/datacontext.html)
implementation per supported database flavor.  For example, for a MySQL
database you would use a
[MySQLDataContext](../../api-doc/latest/classes/mysqldatacontext.html).

In the coming chapters we'll cover instantiating and using this class, but for
now note that the following
[DataContext](../../api-doc/latest/classes/datacontext.html) methods are
commonly used.

* [connect](../../api-doc/latest/classes/datacontext.html#connect): Connect to
  a database..
* [insert](../../api-doc/latest/classes/datacontext.html#insert): Insert a
  [model](../models/) into the database.
* [from](../../api-doc/latest/classes/datacontext.html#from): Select **from**
  the database, or batch update or delete records **from** the database.
* [update](../../api-doc/latest/classes/datacontext.html#update): Update a 
  [model](../models/).
* [delete](../../api-doc/latest/classes/datacontext.html#delete): Delete a 
  [model](../models/).
* [beginTransaction](../../api-doc/latest/classes/datacontext.html#begintransaction):
  Start a database transaction.

Each will be explored in greater detail in the next sections.
