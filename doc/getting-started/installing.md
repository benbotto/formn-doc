---
layout: default
title: Installing
parent: Getting Started
nav_order: 2
---

# Installing

At a minimum, your project will need `formn` and a supported database driver
installed.

### Install Formn

```
npm install --save formn
```

### Install a Database Driver

- MySQL
```
npm install --save mysql2 types/mysql2 
```

### Optional Dependencies

Formn is database first, meaning that you create your database schema first,
and _then_ define entity model classes in TypeScript.  The Formn command-line
interface (CLI) can be used to run database migrations--making and manipulating
tables and such.  The CLI also has a model generator that can be used to
generate entity classes for you.  You can install the CLI locally or globally
(the tutorials assume the latter).

```
npm install -g formn-cli
```

Throughout these tutorials [ts-node](https://github.com/TypeStrong/ts-node) is
used to execute the example TypeScript scripts, so you may want to install
that.

```
npm install --save-dev ts-node
```

Lastly, the tutorials use some of the built-in Node.js utilities, so you might
want to install `@types/node`.

```
npm install --save @types/node
```
