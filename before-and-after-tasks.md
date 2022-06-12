---
layout: default
title: before and after tasks
nav_order: 11
permalink: /before-and-after-tasks
---

# "before" and "after" tasks
{: .no_toc }

1. TOC
{:toc}

## "makes" work flow

The whole [work flow](https://github.com/makesjs/makes/blob/master/lib/index.js) is not complex.

1. download skeleton.
2. read skeleton optional files `questions.js`, `transforms.js`, `before.js`, and `after.js`.
3. runs optional "before" task defined by `before.js`.
4. run through questions, gather [`properties` and `features`](questions/features-and-properties).
5. write out project.
6. runs optional "after" task defined by `after.js`.

## "after" task

We start with the more common "after" task.

"after" task happens after "makes" finished writing out project.

For example you can follow up in a JavaScript project to install npm dependencies.

The [`dumberjs/new` `after.js`](https://github.com/dumberjs/new/blob/master/after.js) is a great example.

> For more information, please review [CommonJS/ESM support](./commonjs-esm).

```js
const {execSync} = require('child_process');

function isAvailable(bin) {
  try {
    execSync(bin + ' -v')
    return true;
  } catch (e) {
    return false;
  }
}

module.exports = async function({
  unattended, here, prompts, run, properties, features, notDefaultFeatures, ansiColors
}) {
  const c = ansiColors;
  let depsInstalled = false;

  if (!unattended) {
    const choices = [
      {title: 'No'},
      {value: 'npm', title: 'Yes, use npm'}
    ];

    if (isAvailable('yarn')) {
      choices.push({value: 'yarn', title: 'Yes, use yarn'});
    }

    if (isAvailable('pnpm')) {
      choices.push({value: 'pnpm', title: 'Yes, use pnpm'});
    }

    const result = await prompts.select({
      message: 'Do you want to install npm dependencies now?',
      choices
    });

    if (result) {
      await run(result, ['install']);
      depsInstalled = true;
    }

    console.log(`\nNext time, you can try to create similar project in silent mode:`);
    console.log(c.inverse(` npx makes dumberjs new-project-name${here ? ' --here' : ''} -s ${notDefaultFeatures.length ? (notDefaultFeatures.join(',') + ' ') : ''}`));
  }

  console.log(`\n${c.underline.bold('Get Started')}`);
  console.log('cd ' + properties.name);
  if (!depsInstalled) console.log('npm install');
  console.log('npm start\n');
};
```

You define a function in `after.js`.

1. the single input argument has following fields:
  * `unattended` is true when in silent mode.
  * `here` is true when in here mode.
  * `features` is the selected features array from select prompts.
  * `notDefaultFeatures` is a sub-set of `features` array, only contains non-default choices. You can use this array to construct a clear silent mode command, as the above example showed.
  * `properties` is the `properties` hash object from text prompts.
  * `prompts` is the exposed "makes" inner prompts implementation. You can call `prompts.text(opts)` and `prompts.select(opts)` to ask user questions, more details in [prompts api doc](prompts-api).
  * `ansiColors` is the exposed ["ansi-colors" npm package](https://www.npmjs.com/package/ansi-colors). This is provided for skeleton to avoid some runtime dependencies.
  * `sisteransi` is the exposed ["sisteransi" npm package](https://www.npmjs.com/package/sisteransi). This is provided for skeleton to avoid some runtime dependencies.
  * `run(cmd, args)` is a convenient function to run command. It runs the command with optional arguments in the final project folder. The above example shows `run('npm', ['install'])`.
2. the result value is ignored. But "after" task can be async (returns a promise).

## "before" task

The optional "before" task happens right before prompting questions, it can:

1. silence the questions defined in `questions.js`. For example, "before" task can let user to choose a preset to skip detailed questionnaire.
2. override preselectedFeatures.
3. override predefinedProperties.
4. or maybe just print out some greetings.

"before" task should also respect existing silent mode (aka unattended).

You define a function in `before.js`.

1. the single input argument has following fields:
  * `unattended` is true when in silent mode.
  * `here` is true when in here mode.
  * `preselectedFeatures` is the pre-selected features array from optional command line switch `-s x,y,z`, default to an empty array.
  * `predefinedProperties` is the pre-defined properties hash object gathered from command line, default to an empty object.
  * `prompts` is the exposed "makes" inner prompts implementation. You can call `prompts.text(opts)` and `prompts.select(opts)` to ask user questions, more details in [prompts api doc](prompts-api).
  * `ansiColors` is the exposed ["ansi-colors" npm package](https://www.npmjs.com/package/ansi-colors). This is provided for skeleton to avoid some runtime dependencies.
  * `sisteransi` is the exposed ["sisteransi" npm package](https://www.npmjs.com/package/sisteransi). This is provided for skeleton to avoid some runtime dependencies.
2. "before" task can be async (returns a promise). The result value is optional, but when you return a value object, it can have following optional fields:
  * `silentQuestions`, if set to `true`, the questions defined in `questions.js` will be silenced.
  * `preselectedFeatures`, overrides existing `preselectedFeatures`.
  * `predefinedProperties`, overrides existing `predefinedProperties`.

For example, aurelia/new skeleton use "before" task to ask user to:
1. pick a preset (silence questionnaire, plus override `preselectedFeatures`).
2. or use custom mode (does nothing, run through questions interactively).

The [`aurelia/new` `before.js`](https://github.com/aurelia/new/blob/master/before.js) is a great example. It asks user to pick a preset, if user picked one, it will skip following questionnaire and overwrite preselectedFeatures.

```js
const PRESETS = {
  'default-esnext': ['webpack', 'babel'],
  'default-typescript': ['webpack', 'typescript'],
};

module.exports = async function({unattended, prompts}) {
  // don't ask when running in silent mode.
  if (unattended) return;

  const preset = await prompts.select({
    message: 'Would you like to use the default setup or customize your choices?',
    choices: [
      {
        value: 'default-esnext',
        title: 'Default ESNext Aurelia 2 App',
        hint: 'A basic Aurelia 2 App with Babel and Webpack'
      }, {
        value: 'default-typescript',
        title: 'Default TypeScript Aurelia 2 App',
        hint: 'A basic Aurelia 2 App with TypeScript and Webpack'
      }, {
        title: 'Custom Aurelia 2 App',
        hint: 'Select bundler, transpiler, and more.'
      }
    ]
  });

  if (preset) {
    const preselectedFeatures = PRESETS[preset];
    if (preselectedFeatures) {
      return {
        silentQuestions: true, // skip following questionnaire
        preselectedFeatures
      };
    }
  }
};
```
