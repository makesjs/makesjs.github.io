---
layout: default
title: Prompts API doc
nav_order: 12
description: Learn how to use the inner prompts API
permalink: /prompts-api
---

# Prompts API doc

The inner prompts API is exposed to [prepend/append transform](transforms), and ["before" and "after" tasks](before-and-after-tasks).

## prompts.text

Use async/await syntax.

```js
const userProvidedText = await prompts.text({message: 'Can you provide some text?'});
```

Or use traditional promise API.

```js
prompts.text({message: 'Can you provide some text?'}).then(
  userProvidedText => {

  },
  err => {
    // aborted, user hit "ESC" key.
  }
);
```

Note unlike the text prompt options in [questions](questions/text), you don't need `name` in the options. This API `prompts.text()` directly returns user input without the need to wrap it into an object `{'field-name': 'user input'}`.

You can supply additional options `default`, `validate` you saw in [questions](questions/text).

## prompts.select

Use async/await syntax.

```js
const result = await prompts.select({
  message: 'Can you choose one of these?',
  choices: [
    {value: 'a', title: 'Something'},
    {value: 'b', title: 'Another thing'}
  ]
});
// result is either 'a' or 'b';
```

Or use traditional promise API.

```js
prompts.select({
  message: 'Can you choose one of these?',
  choices: [
    {value: 'a', title: 'Something'},
    {value: 'b', title: 'Another thing'}
  ]
}).then(
  result => {
    // result is either 'a' or 'b';
  },
  err => {
    // aborted, user hit "ESC" key.
  }
);
```

## Multi-select

Use async/await syntax.

```js
const result = await prompts.select({
  multiple: true,
  message: 'Can you choose one or more of these?',
  choices: [
    {value: 'a', title: 'Something'},
    {value: 'b', title: 'Another thing'}
  ]
});
// result can be [], or ['a'], or ['b'], or ['a', 'b']
```

Or use traditional promise API.

```js
prompts.select({
  multiple: true,
  message: 'Can you choose one or more of these?',
  choices: [
    {value: 'a', title: 'Something'},
    {value: 'b', title: 'Another thing'}
  ]
}).then(
  result => {
    // result can be [], or ['a'], or ['b'], or ['a', 'b']
  },
  err => {
    // aborted, user hit "ESC" key.
  }
);
```
