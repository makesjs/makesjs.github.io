---
layout: default
title: Transforms
nav_order: 9
permalink: /transforms
---

# Transforms
{: .no_toc }

1. TOC
{:toc}

"makes" provided conditional file and write policy, it can cover majority of use cases. But it's not flexible enough to cover all use cases.

For example in [`makesjs/demo2`](https://github.com/makesjs/demo2), there is one `index.js` for `babel`, and another `index.ts` for `typescript`.

```
─ nodejs/
  └── src/
      ├── index.js__if_babel
      └── index.ts__if_typescript
```
{: .lh-tight}

If you check the content of the two files, they are identical. It's kind of waste to have two files. What about to define just one file:

```
─ nodejs/
  └── src/
      └── index.ext
```
{: .lh-tight}

Then somehow changes file name from `index.ext` to `index.js` or `index.ts` based on `features` array? Well, "makes" didn't provide any direct way of doing this, but you can tape into "makes" file objects stream to easily support this feature with just few lines of code.

## "makes" file objects stream

We briefly showed you file objects stream in [feature folders](feature-folders#file-objects-stream). Here is the more complete version of [the stream](https://github.com/makesjs/makes/blob/master/lib/write-project/index.js). Roughly like following gulp stream:

```js
gulp.src(['skeleton/common/**/*', 'skeleton/nodejs/**/*', 'skeleton/babel/**/*'])

  // customise point
  .pipe(customise_prepend_transforms)

  .pipe(mark_write_policy)
  .pipe(filter_conditional_file_and_folder)
  .pipe(preprocess_file_content)
  .pipe(merge_readme_and_json) // To be explained

  // customise point
  .pipe(customise_append_transforms)

  .pipe(check_project_folder_and_honour_write_policy)
  .pipe(gulp.dest(project_folder));
```

There are two customise points for skeleton designer to tape into the stream, "prepend" and "append" transforms.

## Default behaviour on duplicated skeleton files

Before show you the custom transforms, we need to explain more on step `merge_readme_and_json`.

When you define duplicated skeleton files. For example:

```
common/a-file
nodejs/a-file
```

> "makes" default behaviour is to honour the last one `nodejs/a-file`, so the order of `features` array is actually significant. You can deliberately offer a new file in some feature folder which overshadows previous feature folders (according to the order of `features` array, which is the order of skeleton questions and choices).

However, for convenience, "makes" offers special behaviour on two types of files.

### 1. Concatenate readme files

If the duplicated skeleton files is a readme file (`/readme(\.(md|txt|markdown))?$/i`), "makes" will concatenate them together. Note for convenience, "makes" appends a new line before appending the content of additional readme.

If you have `common/README.md` with content `a`, `nodejs/README.md` with content `b`, the final `README.md` could be `a\nb` if end user selected `nodejs`.

### 2. Merge JSON files

For simpler skeleton of JavaScript projects, "makes" merges JSON files (which includes `package.json`) from duplicated files into one.

```
common/package.json
nodejs/package.json
nodejs/package.json__if_babel
```

"makes" will merge the three `package.json` into one if end user selected `nodejs` and `babel`.

"makes" not only merges JSON files, but also cleans it up. So you can write:

```json
{
  // @if a
  "a": true,
  // @endif
  // @if b
  "b": true
  // @endif
}
```

Without worrying about the trailing `,` in `{"a": true,}`.

## Append transforms

We will exam "append" transforms first because it's the more common than "prepend" transforms.

[`makesjs/demo2#adv-through2` `transforms.js`](https://github.com/makesjs/demo2/blob/adv-through2/transforms.js) implemented an "append" transform that translate `file.ext` to `file.js`/`file.ext` based on `features` array.

You can try this demo2 branch with `npx makes makesjs/demo2#adv-through2`.

Create this optional `transform.js` file.

Here is an example code in CommonJS format.

> For more information, please review [CommonJS/ESM support](./commonjs-esm).

```js
const through2 = require('through2');

exports.append = function(properties, features) {
  return through2.obj(function(file, env, cb) {
    if (file.isBuffer() && file.extname === '.ext') {
      // change .ext to .ts or .js file
      file.extname = features.includes('typescript') ? '.ts' : '.js'
    }
    cb(null, file);
  });
};
```

For users who has experience with gulp, this is easy to understand.

The `exports.append` can be one function or array of functions.

Every function:
* get five input arguments `properties`, `features`, `targetDir`, `unattended`, and `prompts`.
  * `properties`, `features` are the result built from questions.
  * `targetDir` gives you a chance to inspect target folder, obviously only useful when end users ran "makes" in [here mode](here-mode-and-write-policy).
  * `unattended` is true when end users ran "makes" in [silient mode](silent-mode). Your transform implementation should skip any user interactivity when `unattended` is true.
  * `prompts` is the exposed "makes" inner prompts implementation. You can call `prompts.text(opts)` and `prompts.select(opts)` to ask user questions, more details in [prompts api doc](prompts-api).
* return a Node.js transform stream. Here we use `through2` to generate a transform stream, just like what you would see in any gulp tutorial.

## Runtime dependencies

The above "append" transform imposed a runtime dependency on `through2`. To tell "makes" that your skeleton needs additional npm package, you need to create the optional `package.json` file in your skeleton with `"dependencies"`.

[`makesjs/demo2#adv-through2` `package.json`](https://github.com/makesjs/demo2/blob/adv-through2/package.json)

```json
{
  "dependencies": {
    "through2": "^3.0.1"
  }
}
```

When "makes" loads up this skeleton, it sees non-empty `"dependencies"`, it then fires up `npm install --only=prod` to install all the dependencies before proceed.

> `"devDependencies"` is irrelevant. You can add many npm packages to `"devDependencies"` to help local testing or changelog, it would not slow "makes" down.

### Aim zero runtime dependency

There is no doubt that it will slow down "makes" to install `through2`. Ideally you should aim zero runtime dependency.

For example, `through2` is absolutely not needed to create a transform stream. The modern Node.js API is simple enough to create a transform stream.

You can replace:

```js
const through2 = require('through2');
through2.obj(
  function(file, env, cb) {
    // ...
  },
  function(cb) {
    // optional flush
  }
);
```

With
```js
const {Transform} = require('stream'); // stream is core Node.js module
new Transform({
  objectMode: true,
  transform: function(file, enc, cb) {
    // ...
  },
  flush: function(cb) {
    // optional flush
  }
});
```

The other demo2 branch [`makesjs/demo2#adv` `transforms.js`](https://github.com/makesjs/demo2/blob/adv/transforms.js) implemented exact same append transform without using through2.

```js
const {Transform} = require('stream');

exports.append = function(properties, features) {
  return new Transform({
    objectMode: true,
    transform: function(file, env, cb) {
      if (file.isBuffer() && file.extname === '.ext') {
        // change .ext to .ts or .js file
        file.extname = features.includes('typescript') ? '.ts' : '.js'
      }
      cb(null, file);
    }
  });
};
```

The benefit is `npx makes makesjs/demo2#adv` is faster than `npx makes makesjs/demo2#adv-through2` without the need to install runtime dependencies.

> "makes" itself uses [@vercel/ncc](https://github.com/vercel/ncc) to ship npm package "makes" in a bundle without any additional runtime dependencies, that's part of the reason why `npx makes <skeleton_provider>` is so fast.

## Prepend transforms

Prepend transform is for advanced use cases. Different from append transform which sees [vinyl file](https://github.com/gulpjs/vinyl) after the main stages of "makes" stream pipeline, prepend transform is before "makes" did anything.

While append transform always sees processed vinyl file like `index.js`, prepend transform can see the original vinyl file like `index.js__skip-if-exits__if_babel`. We can use prepend transform to add additional meta data to vinyl file object, then pair with some append transform to reason about those meta data before final write-out.

We don't have concrete examples now, but we will add some in future.

TODO: add example from future aurelia skeleton how to use prepend/append transform to print out dotnet-core instructions.

