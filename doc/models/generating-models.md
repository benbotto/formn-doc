---
layout: default
title: Generating Models
parent: Models
nav_order: 3
---

# Generating Models

Manually defining models is cumbersome, so Formn's CLI provides a helper tool
to generate models from an existing database schema.

### Prerequisites

If you haven't already
done so, install the `formn-cli` package globally.

```
npm install -g formn-cli
```

You'll also need to define a `connections.json` file as described in the
[Connecting](../connecting/) section.  Formn generates models using database
metadata defined in the `INFORMATION_SCHEMA` tables, so the credentials
supplied in the `connections.json` file must have sufficient permission.

### Generating Models

The Formn CLI command for generating models is `generate`, aliased `g`.  You
can see all the commands and options using the `--help` switch.

```
formn --help
formn generate --help
```

Options include:

* `--connections-file`, aliased as `-c`.  The path to a `connections.json`
  file, which defaults to `./connections.json`
* `--flavor`, aliased `-f`.  The database type, which defaults to `mysql`.

The `generate` command requires one argument: the path where model classes
should be written.  Here's an example.

```
formn g src/entity/
```

Assuming you've been following along with the tutorial, running the above
command will generate two model classes:
[person.entity.ts](https://github.com/benbotto/formn-example/blob/master/src/entity/person.entity.ts)
and
[phone-number.entity.ts](https://github.com/benbotto/formn-example/blob/master/src/entity/phone-number.entity.ts).

### Customizing Generated Models

You may have noticed that the class and property names in the generated models
do not correspond directly to the database table and column names.  For
example, the class names are singular and pascal case (`Person` and
`PhoneNumber`), whereas the database tables names are plural and snake case
(`people` and `phone_numbers`).  Also, the properties associated with primary
keys have been aliased as `Person.id` and `PhoneNumber.id`.  If that naming
convention is okay with you, then feel free to skip this section, but do note
that the formatting can be customized to your liking.

The code responsible for model generation in Formn is the
[ModelGenerator](../../api-doc/latest/classes/modelgenerator.html) class.
(There's one implementation per database flavor, for example
[MySQLModelGenerator](../../api-doc/latest/classes/mysqlmodelgenerator.html)).
The [ModelGenerator
constructor](../../api-doc/latest/classes/modelgenerator.html#constructor)
takes three parameters that are used for formatting class names,
column-decorated property names, and relationship-decorated property names.
Those parameters are instances of
[TableFormatter](../../api-doc/latest/interfaces/tableformatter.html),
[ColumnFormatter](../../api-doc/latest/interfaces/columnformatter.html), and
[RelationshipFormatter](../../api-doc/latest/interfaces/relationshipformatter.html),
respectively.  The Formn CLI uses the defaults:
[DefaultTableFormatter](../../api-doc/latest/interfaces/defaulttableformatter.html),
[DefaultColumnFormatter](../../api-doc/latest/interfaces/defaultcolumnformatter.html),
and
[DefaultRelationshipFormatter](../../api-doc/latest/interfaces/defaultrelationshipformatter.html).

If you want to customize the generated models, then you'll need to implement
one or more of the above interfaces, and then instantiate a
[ModelGenerator](../../api-doc/latest/classes/modelgenerator.html) manually.
Take a look at the formn-cli code for an example:
[cli-model-generator.ts](https://github.com/benbotto/formn-cli/blob/1.0.0/src/lib/cli-model-generator.ts#L22).
