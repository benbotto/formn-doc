---
layout: default
title: TypeScript Config
parent: Getting Started
nav_order: 1
---

# TypeScript Config

Formn is intended for TypeScript projects that run on Node.js, and it requires
decorators and decorator metadata to be enabled.  In your project, you'll need
to enable these configuration settings in your
[`tsconfig.json`](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
file.  A typical, working `tsconfig.json` file is presented below.

```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "noImplicitAny": true,
    "noImplicitThis": true,
    "strictPropertyInitialization": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "rootDir": "./src",
    "outDir": "./dist"
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

The `tsconfig.json` file should be placed at the root of your project.  In the
above example, the project's code is expected to reside in the `src` directory,
and the built code will be output to the `dist` folder.
