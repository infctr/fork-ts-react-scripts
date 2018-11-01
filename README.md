# fork-ts-react-scripts

A fork of CRA 2.1 that doesn't mess up with `tsconfig.json` absolute paths import feature.

## How it works

Original CRA with typescript support overrides `paths` and `baseUrl` settings in an existing typescript project enforcing to use relative paths imports ([refer](https://github.com/facebook/create-react-app/issues/5585#issuecomment-433751395)) which pretty much sucks. This version of `react-scripts` removes these strict checks allowing to keep `paths` in `tsconfig.json` just to keep the changes minimal.

## Usage

In order to make absolute path imports work webpack config has to be tweaked just a tad bit with the help of [arackaf/customize-cra](https://github.com/arackaf/customize-cra) package and [timarney/react-app-rewired](https://github.com/timarney/react-app-rewired). Install these two and create a `config-overrides.js` file at the root of your project. Add aliases and module directories that you like for webpack `resolve` settings:

```js
// config-overrides.js
const path = require('path');
const {
  override,
  addWebpackAlias,
  addBundleVisualizer,
} = require('customize-cra');

const addWebpackModules = modules => config => {
  if (!config.resolve) {
    config.resolve = {};
  }

  if (!config.resolve.modules) {
    config.resolve.modules = ['node_modules'];
  }

  config.resolve.modules = [...config.resolve.modules, ...modules];

  return config;
};

module.exports = override(
  process.env.BUNDLE_VISUALIZE && addBundleVisualizer(),
  addWebpackModules([path.resolve(__dirname, 'src')]),
  addWebpackAlias({
    ['scripts']: path.resolve(__dirname, 'scripts'),
    ['e2e']: path.resolve(__dirname, 'e2e'),
  })
);
```

Update your `package.json` tasks to make sure CRA's config gets "rewired":

```diff
"scripts": {
-  "start": "react-scripts start",
+  "start": "react-app-rewired start --scripts-version fork-ts-react-scripts",
-  "build": "react-scripts build"
+  "build": "react-app-rewired build --scripts-version fork-ts-react-scripts"
}
```

And that's it! Now you can have your cake and eat it 🎉.

## Tracking changes from the CRA upstream

Forking a single package from a monorepo is not as straightforward as it might seem, so it was created as a git subtree of the original CRA with squashed commits preserving the ability to track upstream changes. Refer to this [gist](https://gist.github.com/alfredringstad/ac0f7a1e081e9ee485e653b6a8351212) for some guidance.
