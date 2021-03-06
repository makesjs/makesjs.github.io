---
layout: default
title: '@eval'
nav_order: 3
permalink: /preprocess-file-content/eval
parent: Preprocess File Content
---

# @eval

`@eval` is brought in because `@echo` is not enough. For example in a json file:

```json
{
  "description": "/* @echo description */"
}
```

What if `description` contains some character like `"` which needs to escaped? `@echo` is not flexible to deal with such situation.

You can eval a JavaScript expression involving one or more properties from the [`properties`](../questions/features-and-properties#properties) hash object.

All following examples are evaluated against these properties:

```js
var properties = {
  name: 'my-app',
  description: '"makes" is great!',
  title: '<makes>'
  author: 'CP'
};
```

## Use @eval to escape a string in JavaScript/JSON

We can borrow `JSON.stringify()` for this task, note the result already wrapped with double-quotes.

```json
{
  "name": "/* @echo name */",
  "description": /* @eval JSON.stringify(description) */
}
```

Yield result:

```json
{
  "name": "my-app",
  "description": "\"makes\" is great!"
}
```

## Echo project name

We used `@echo name` to display our project name in many examples, you might noticed we even used it inside JSON file without worrying about string escaping.

> Echo project name is safe, as long as you didn't modify the default validation for project name (default validation only allows letters, numbers, dash(`-`) and underscore(`_`) in project name).

## Use @eval to escape a string in HTML

JavaScript didn't provide any built-in function to escape string in HTML. We have to use this verbose expression.

```js
aStr.replace(/[&<>"']/g, char => ({'&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'})[char])
```

```html
<div>
  <p><!-- @eval (title || '').replace(/[&<>"']/g, char => ({'&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'})[char]) --></p>
</div>
```

Yield result:

```html
<div>
  <p>&lt;makes&gt;</p>
</div>
```

## Other usage

You can use `@eval` to cover more situations beyond escaping. It's flexible but dangerous, use it rarely.

For example, you can print out current year in license file like this:

```
Copyright (c) /* @eval new Date().getFullYear() */ /* @echo author */
```

## Enhance eval

"makes" doesn't provide any API to enhance the available functions that you can use inside `@eval`. It's understandable that you feel it's too tedious to escape a string in HTML with `@eval`.

However, you can enhance eval without any help from "makes", because JavaScript allows monkey-patch :-)

There is a [demo](https://github.com/makesjs/demo3-enhance-eval) for it!

    npx makes makesjs/demo3-enhance-eval

This demo shows how to inject a function as Nodejs global variable
in the before task (before.js).

```js
global.escapeHTML = require('escape-html');
```

> Note this demo skeleton has a `package.json` file with `escape-html`
in the `"dependencies"`, not `"devDependencies"`. Because it requires
that npm module at runtime.

Because before.js is loaded before preprocessing all skeleton files,
this global function is then available in the @eval directive.

It's then used in common/index.html as

```html
<h1><!--  @eval escapeHTML(name) --></h1>
<p><!-- @eval escapeHTML(description) --></p>
```