---
layout: default
title: Home
nav_order: 1
permalink: /
---

# makes
{: .fs-10 }

A tool to scaffold new projects, simple enough that you would want to define your own skeletons (aka generators).
{: .fs-6 .fw-300 }

[Get Started](./get-started){: .btn .btn-blue .mr-3 .fs-5 } [View on Github](https://github.com/makesjs/makes){: .btn .fs-5 }

---

## Run makes

As long as you have [`Node.js`](https://nodejs.org) installed, you can run `makes` without any installation.

```bash
npx makes <skeleton_provider>
```

![makes dumberjs]({{ site.baseurl }}/assets/makes-dumberjs.gif)
{: .mx-lg-8 .mx-md-6 }

## Example skeletons

### [`aurelia/new`](https://github.com/aurelia/new)

Skeletons for upcoming [Aurelia 2](https://docs.aurelia.io). Currently in early-alpha stage.

```bash
npx makes aurelia
```

### [`dumberjs/new`](https://github.com/dumberjs/new)

Try `dumberjs` skeleton to create various types of front-end projects. [`dumberjs`](https://github.com/dumberjs/dumber) is a JavaScript bundler using AMD module format for front-end SPA apps.

```bash
npx makes dumberjs
```

> `npx makes dumberjs` is a conventional short-cut of `npx makes dumberjs/new`.

### [`makesjs/demo1`](https://github.com/makesjs/demo1)

A skeleton demo for plain simple one file project.

```bash
npx makes makesjs/demo1
```

### [`makesjs/demo2`](https://github.com/makesjs/demo2)

A skeleton demo for customised questions and feature folders.
```bash
npx makes makesjs/demo2
```

## License

"makes" is licensed under the [MIT license](https://github.com/makesjs/makes/blob/master/LICENSE).

## Acknowledgements

"makes" borrowed code from [prompts](https://github.com/terkelg/prompts), [preprocess](https://github.com/jsoverson/preprocess), and [aurelia-cli](https://github.com/aurelia/cli).
