---
layout: default
title: CommonJS/ESM Support
nav_order: 14
permalink: /commonjs-esm
---

# CommonJS/ESM Support

"makes" supports both CommonJS and ESM code in all the customisable files:
* `questions.js`
* `transforms.js`
* `before.js`
* `after.js`

The format is handled by Nodejs itself, not "makes".
* CommonJS: `.cjs` files or plain `.js` files.
* ESM: `.mjs` files or `.js` files with additional `package.json`'s `"type": "module"`.

For more information on Nodejs' support of ESM, please review Nodejs doc [Determining module system](https://nodejs.org/dist/latest-v18.x/docs/api/packages.html#determining-module-system).

> When using 3rd party libs, the CommonJS might be easier to work with, as many 3rd party libs only ships CommonJS code, such as various stream related libs.

## Example skeleton [`makesjs/demo3-enhance-eval` esm branch](https://github.com/makesjs/demo3-enhance-eval/tree/esm)

```bash
npx makes makesjs/demo3-enhance-eval#esm
```

## `questions.js`

To use CommonJS format, use file name `questions.js` or `questions.cjs` with following content:

```js
// CommmonJS
module.exports = [
  // list of questions...
];
```

To use ESM format, use file name `questions.mjs` or `questions.js` with additional `package.json` with `"type": "module"` (so that Nodejs will treat any JS file in your skeleton with ESM format):

`questions.mjs`
```js
// ESM
export default [
  // list of questions ...
]
```

`package.json`, only needed if you use file name `questions.js` for ESM code.
```json
{ "type": "module" }
```

## `transforms.js`

Transforms doesn't utilise default export, it expects two optional exports: `append` and `prepend`.

```js
// Commonjs
exports.append = a_stream_transform;
// or exports.append = [ a list of transforms ];
exports.prepend = a_stream_transform;
// or exports.prepend = [ a list of transforms ];
```

```js
// ESM
const append = a_stream_transform;
// or const append = [ a list of transforms ];
const prepend = a_stream_transform;
// or const prepend = [ a list of transforms ];
export { append, prepend };
```

## `before.js` and `after.js`

```js
// CommmonJS
module.exports = async function (...) {...}
```

```js
// ESM
export default async function (...) {...}
```


