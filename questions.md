---
layout: default
title: Questions
nav_order: 4
permalink: /questions
has_children: true
---

# Questions

We will use [`makesjs/demo2`](https://github.com/makesjs/demo2). Try it with:

```bash
npx makes makesjs/demo2 # or npx makes makesjs/demo2 my-app
```

![makes-demo2 screenshot]({{ site.baseurl }}/assets/makes-demo2.gif)

This demo skeleton defined an optional file [`questions.js`](https://github.com/makesjs/demo2/blob/master/questions.js), plus some [feature folders](feature-folders) `nodejs` and `ruby` (we will talk about them in next page).

```
─ questions.js
─ common/
  ├── LICENSE__if_license
  └── REAME.md
─ nodejs/
  ├── .babelrc__if_babel
  ├── .gitignore
  ├── index.js__if_not_babel_and_not_typescript
  ├── package.json
  └── src/
      ├── index.js__if_babel
      └── index.ts__if_typescript
─ ruby/
  └── main.rb
```
{: .lh-tight}

The optional `questions.js` file needs to provide an array of questions. "makes" support CommonJS and ESM. For simplicity, "makes" doesn't support any Babel/TypeScript transpiling. It must be written in pain CommonJS/ESM format which Node.js can understand.

To use CommonJS format, use file name `questions.js` or `questions.cjs` with following content:

```js
module.exports = [
  // list of questions...
];
```

To use ESM format, use file name `questions.mjs` or `questions.js` with additional `package.json` with `"type": "module"` (so that Nodejs will treat any JS file in your skeleton with ESM format):

`questions.mjs`
```js
export default [
  // list of questions ...
]
```

`package.json`, only needed if you use file name `questions.js` for ESM code.
```json
{ "type": "module" }
```

For more information, please review [CommonJS/ESM support](./commonjs-esm).

"makes" supports only three types of questions: text prompt, select prompt, multi-select prompt. These prompts are based on [prompts](https://github.com/terkelg/prompts), but customised and simplified.
