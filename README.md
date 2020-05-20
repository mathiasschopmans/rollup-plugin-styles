# rollup-plugin-styles

[![npm version](https://img.shields.io/npm/v/rollup-plugin-styles)](https://www.npmjs.com/package/rollup-plugin-styles)
[![monthly downloads count](https://img.shields.io/npm/dm/rollup-plugin-styles)](https://www.npmjs.com/package/rollup-plugin-styles)
[![required rollup version](https://img.shields.io/npm/dependency-version/rollup-plugin-styles/peer/rollup)](https://www.npmjs.com/package/rollup)
[![dependencies status](https://img.shields.io/david/Anidetrix/rollup-plugin-styles)](https://david-dm.org/Anidetrix/rollup-plugin-styles)
[![build status](https://github.com/Anidetrix/rollup-plugin-styles/workflows/CI/badge.svg)](https://github.com/Anidetrix/rollup-plugin-styles/actions?query=workflow%3ACI)
[![code coverage](https://codecov.io/gh/Anidetrix/rollup-plugin-styles/branch/master/graph/badge.svg)](https://codecov.io/gh/Anidetrix/rollup-plugin-styles)
[![license](https://img.shields.io/github/license/Anidetrix/rollup-plugin-styles)](./LICENSE)

### 🎨 Universal [Rollup](https://github.com/rollup/rollup) plugin for styles:

- [PostCSS](https://github.com/postcss/postcss)
- [Sass](https://github.com/sass/dart-sass)
- [Less](https://github.com/less/less.js)
- [Stylus](https://github.com/stylus/stylus)
- [CSS Modules](https://github.com/css-modules/css-modules)
- URL resolving/rewriting with asset handling
- Ability to use `@import` statements inside regular CSS

...and much more!

## Table of Contents

- [Installation](#installation)
- [Usage](#usage)
  - [Importing a file](#importing-a-file)
    - [CSS/Stylus](#cssstylus)
    - [Sass/Less](#sassless)
  - [CSS Injection](#css-injection)
  - [CSS Extraction](#css-extraction)
  - [Emitting processed CSS](#emitting-processed-css)
  - [CSS Modules](#css-modules)
  - [With Sass/Less/Stylus](#with-sasslessstylus)
  - [`fibers` (**Sass only**)](#fibers-sass-only)
- [Configuration](#configuration)
- [Main differences from `rollup-plugin-postcss`](#main-differences-from-rollup-plugin-postcss)
- [Contributing](#contributing)
- [License](#license)
- [Thanks](#thanks)

## Installation

```bash
npm install -D rollup-plugin-styles # npm

pnpm add -D rollup-plugin-styles # pnpm

yarn add rollup-plugin-styles --dev # yarn 1.x
```

## Usage

```js
// rollup.config.js
import styles from "rollup-plugin-styles";

export default {
  output: {
    // Governs names of CSS files (for assets from CSS use `hash` option for url handler).
    // Note: using value below will put .css files near js,
    // but make sure to adjust `hash`, `assetDir` and `publicPath`
    // options for url handler accordingly.
    assetFileNames: "[name]-[hash][extname]",
  },
  plugins: [styles()],
};
```

After that you can import CSS files in your code:

```js
import "./style.css";
```

Default mode is `inject`, which means CSS is embedded inside JS and injected into `<head>` at runtime, with ability to pass options to CSS injector or even pass your own injector.

CSS is available as default export in `inject` and `extract` modes, but if [CSS Modules](https://github.com/css-modules/css-modules) are enabled you need to use named `css` export.

```js
// Injects CSS, also available as `style` in this example
import style from "./style.css";
// Named export of CSS string
import { css } from "./style.css";
```

In `emit` mode none of the exports are available as CSS is purely processed and passed along the build pipeline, which is useful if you want to preprocess CSS before using it with CSS consuming plugins, e.g. [rollup-plugin-lit-css](https://github.com/bennypowers/rollup-plugin-lit-css).

PostCSS configuration files will be found and loaded automatically, but this behavior is configurable using `config` option.

### Importing a file

#### CSS/Stylus

```css
/* Import from `node_modules` */
@import "bulma/css/bulma";
/* Local import */
@import "./custom";
/* ...or (if no package named `custom` in `node_modules`) */
@import "custom";
```

#### Sass/Less

You can prepend the path with `~` to resolve in `node_modules`:

```scss
// Import from `node_modules`
@import "~bulma/css/bulma";
// Local import
@import "custom";
// ...or
@import "./custom";
```

Also note that partials are considered first, e.g.

```scss
@import "custom";
```

Will look for `_custom` first _(with the approptiate extension(s))_, and then for `custom` if `_custom` doesn't exist.

### CSS Injection

```js
styles({
  mode: "inject", // Unnecessary, set by default
  // ...or with custom options for injector
  mode: ["inject", { container: "body", singleTag: true, prepend: true }],
  // ...or with custom injector
  mode: ["inject", yourInjectorFn],
});
```

### CSS Extraction

```js
styles({
  mode: "extract",
  // ... or with relative to output dir/output file's basedir (but not outside of it)
  mode: ["extract", "awesome-bundle.css"],
});
```

### Emitting processed CSS

```js
// rollup.config.js
import styles from "rollup-plugin-styles";

// Any plugin which consumes pure CSS
import litcss from "rollup-plugin-lit-css";

export default {
  plugins: [
    styles({ mode: "emit" }),

    // Make sure to list it after this one
    litcss(),
  ],
};
```

### [CSS Modules](https://github.com/css-modules/css-modules)

```js
styles({
  modules: true,
  // ...or with custom options
  modules: {},
  // ...additionally using autoModules
  autoModules: true,
  // ...with custom regex
  autoModules: /\.mod\.\S+$/,
  // ...or custom function
  autoModules: id => id.includes(".modular."),
});
```

### With Sass/Less/Stylus

Install corresponding dependency:

- For `Sass` support install `node-sass` or `sass`:

  ```bash
  npm install -D node-sass # npm

  pnpm add -D node-sass # pnpm

  yarn add node-sass --dev # yarn 1.x
  ```

  ```bash
  npm install -D sass # npm

  pnpm add -D sass # pnpm

  yarn add sass --dev # yarn 1.x
  ```

- For `Less` support install `less`:

  ```bash
  npm install -D less # npm

  pnpm add -D less # pnpm

  yarn add less --dev # yarn 1.x
  ```

- For `Stylus` support install `stylus`:

  ```bash
  npm install -D stylus # npm

  pnpm add -D stylus # pnpm

  yarn add stylus --dev # yarn 1.x
  ```

That's it, now you can import `.scss` `.sass` `.less` `.styl` `.stylus` files in your code.

### `fibers` (**Sass only**)

By default, `fibers` package will be loaded automatically if available when using `sass` implementation.

> When installed via npm, `Dart Sass` supports a JavaScript API that's fully compatible with `Node Sass` <...>, with support for both the render() and renderSync() functions. <...>
>
> Note however that by default, **renderSync() is more than twice as fast as render()** due to the overhead of asynchronous callbacks. To avoid this performance hit, render() can use the `fibers` package to call asynchronous importers from the synchronous code path.
>
> [Source](https://github.com/sass/dart-sass/blob/master/README.md#javascript-api)

To install `fibers`:

```bash
npm install -D fibers # npm

pnpm add -D fibers # pnpm

yarn add fibers --dev # yarn 1.x
```

## Configuration

See [API Reference for `Options`](https://anidetrix.github.io/rollup-plugin-styles/interfaces/_index_d_.options.html) for full list of available options.

## Main differences from [rollup-plugin-postcss](https://github.com/egoist/rollup-plugin-postcss)

- Written completely in TypeScript
- Up-to-date [CSS Modules](https://github.com/css-modules/css-modules) implementation
- Built-in `@import` handler
- Built-in assets handler
- Respects `output.assetFileNames`
- Code splitting support
- Multi entry support
- Ability to emit pure CSS for other plugins
- Correct multiple instance support with check for already processed files
- Support for implementation and `fibers` forcing for Sass
- Support for partials and `~` in Less import statements
- Sourcemaps include source content
- Proper sourcemap generation for all loaders
- Correct inline sourcemaps
- Correct relative source paths in extracted sourcemaps
- Extracts sourcemaps from loaded files
- More smaller things that I forgot

## Contributing

Any contributions are always welcome, not only Pull Requests! 😀

- **QA**: file bug reports, the more details you can give the better
- **Code**: take a look at the [open issues](https://github.com/Anidetrix/rollup-plugin-styles/issues), even if you can't write code showing that you care about a given issue matters
- **Ideas**: feature requests are welcome, even ambitious ones
- **Donations**: financial support helps to dedicate more time to this project

Your First Contribution? You can learn how from this _free_ series, [How to Contribute to an Open Source Project on GitHub](https://egghead.io/series/how-to-contribute-to-an-open-source-project-on-github).

## License

MIT &copy; [Anton Kudryavtsev](https://github.com/Anidetrix)

## Thanks

- [rollup-plugin-postcss](https://github.com/egoist/rollup-plugin-postcss) - for good reference 👍
- [rollup](https://github.com/rollup/rollup) - for awesome bundler 😎
