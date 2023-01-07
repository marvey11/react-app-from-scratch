# React App From Scratch

Setting up a Web App using TypeScript, React, Jest, Babel, Webpack, Babel, ESLint, Prettier, and yarn (classic).

**NOT** going to use `create-react-app` (CRA).

This project may be viewed as a how-to, but in the first place it is a way for me to learn (and document what I have
learned) about what is going on under the hood and what is actually necessary in terms of infrastructure in one of these
projects. But I certainly don't know all the answers.

## Initial Infrastructure

Create a new project folder and change into it:

```bash
mkdir -p react-app-from-scratch
cd react-app-from-scratch
```

Initialise the local `git` repository:

```bash
git init
```

Make sure that `node` is installed on the system. In addition, install `yarn` if it's not already present on your
system:

```bash
npm install --global yarn
```

Create a skeleton `package.json`:

```json
{
  "private": "true",
  "name": "react-app-from-scratch",
  "version": "0.0.1",
  "license": "MIT",
  "scripts": {},
  "dependencies": {},
  "devDependencies": {}
}
```

Run the initial installation. This will create the `node_modules` folder and the `yarn.lock` file.

```bash
yarn install
```

Since we don't want the `node_modules` folder checked in, let's add it to the `.gitignore`

```bash
echo "node_modules/" > .gitignore
```

## Installing and Setting Up Basic Packages

Let's start with TypeScript. Since, at build time, all the TypeScript code is transpiled to JavaScript, we will add
TypeScript in the `devDependencies` only. Let's also add the typings for the Node libraries.

```bash
yarn add --dev typescript @types/node
```

We can now set up TypeScript. Create the following basic `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "outDir": "build",
    "target": "ESNext",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "module": "es6",
    "moduleResolution": "Node",
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true
  }
}
```

Let's create the `src` folder (since we've already specified it in `tsconfig.json`) and the first code file:

```bash
mkdir src
echo 'console.log("Hello, world!");' > src/index.ts
```

Now we can add the `build` script to `package.json`:

```json
{
  "scripts": {
    "build": "tsc"
  }
}
```

That means we're now able to run the build script:

```bash
yarn run build
```

When all goes well, we should see the `build` folder as specified in `tsconfig.json` under `outDir`. If you didn't
specify the `outDir` parameter, the resulting `index.js` will be written to `src` instead.

We should now add the `build` folder to `.gitignore`:

```bash
echo "/build/" >> .gitignore
```

Next, we'll install some additional development tools, specifically `eslint` and `prettier` and assorted plugins.

```bash
yarn add --dev eslint prettier eslint-config-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

Create `.prettierrc.json` according to your preferences:

```json
{
  "printWidth": 120,
  "proseWrap": "always",
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "none"
}
```

We will also need a configuration for `eslint`. This file is called `.eslintrc.json`:

```json
{
  "env": {
    "es2021": true
  },
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "prettier"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "plugins": ["@typescript-eslint"],
  "settings": {},
  "rules": {},
  "reportUnusedDisableDirectives": true
}
```

We can now add linting rules to the `scripts` section in `package.json`:

```json
{
  "scripts": {
    "build": "tsc",
    "lint": "eslint --color --ext .ts --ext .tsx src/",
    "lint:fix": "eslint --color --fix --ext .ts --ext .tsx src/"
  }
}
```

## Babel

Let's add Babel to the mix. The idea is that Babel will be responsible for transpiling the TypeScript code to JavaScript
and `tsc` will be responsible for type-checking.

```
yarn add --dev @babel/core @babel/cli @babel/preset-env @babel/preset-typescript
```

Create the file `babel.config.json` with the following contents:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

We'll also have to adapt `tsconfig.json` since we need to make sure that `tsc` will not be creating `*.js` files any
longer. Add the following lines to the `compilerOptions` section in `tsconfig.json`:

```json
"noEmit": true,
"isolatedModules": true
```

The `noEmit` parameter tells `tsc` not to emit anything any longer. Babel will take care of that. The `isolatedModules`
parameter -- according to the TypeScript documentation
[(link)](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html) -- ensures that Babel can safely
transpile files in the TypeScript project.

And finally, we'll need to change the build rules in `package.json`:

```json
{
  "scripts": {
    "tsc": "tsc",
    "build": "babel src --out-dir build --extensions .ts",
    "lint": "eslint --color --ext .ts --ext .tsx src/",
    "lint:fix": "eslint --color --fix --ext .ts --ext .tsx src/"
  }
}
```

We can now run the build again:

```ShellSession
> yarn run build
yarn run v1.22.19
$ babel src --out-dir build --extensions .ts
Successfully compiled 1 file with Babel (582ms).
Done in 1.27s.
```

As can be seen, the build now utilises Babel.

We should also check that running `tsc` really doesn't emit anything.

```ShellSession
> yarn run tsc
yarn run v1.22.19
$ tsc
src/index.ts:1:1 - error TS1208: 'index.ts' cannot be compiled under '--isolatedModules' because it is considered a global script file. Add an import, export, or an empty 'export {}' statement to make it a module.

1 console.log("Hello, world!");
  ~~~~~~~


Found 1 error in src/index.ts:1

error Command failed with exit code 2.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

Oops! That's apparently a side effect of using `isolatedModules`. Let's do as the message suggests and change `index.ts`
to include an empty `export` statement:

```typescript
console.log("Hello, world!");

export {};
```

Now, let's run `tsc` again:

```ShellSession
> yarn run tsc
yarn run v1.22.19
$ tsc
Done in 2.91s.
```

Perfect!

If you did actually delete the build folder -- just to check, of course -- you will see now that running `tsc` didn't
actually create anything. It is now also safe to remove the following line from `tsconfig.json`:

```json
"outDir": "build"
```

## Webpack

Install the required packages:

```bash
yarn add --dev webpack webpack-cli babel-loader
```

The next step is to configure Webpack. As a first step we'll run the Babel transpiling from Webpack. Create the config
file named `webpack.config.js` with the following contents:

```javascript
const path = require("path");

module.exports = {
  mode: "production",
  entry: "./src/index.ts",
  output: {
    path: path.resolve(__dirname, "./build"),
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.(js|ts)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  },
  resolve: {
    extensions: [".js", ".ts"]
  }
};
```

The VS Code editor complains with something like `'require' is not defined. eslint(no-undef)`. One possible solution is
to add the Webpack configuration to `.eslintignore`:

```bash
echo "webpack.config.js" > .eslintignore
```

An alternative might also be use use TypeScript as the configuration language (see the
[Webpack Documentation](https://webpack.js.org/configuration/configuration-languages/#typescript) for more details.

We can now replace the `build` configuration in `package.json` with the Webpack equivalent. Webpack will delegate
transpiling the TypeScript code to Babel, as defined in `webpack.config.json`. The new `build` configuration is now:

```json
"build": "webpack --progress"
```

With this we don't need the `@babel/cli` package any longer and can remove it:

```bash
yarn remove @babel/cli
```

Running the `build` phase now looks like this:

```ShellSession
> yarn run build
yarn run v1.22.19
$ webpack --progress
asset bundle.js 52 bytes [compared for emit] [minimized] (name: main)
./src/index.ts 40 bytes [built] [code generated]
webpack 5.75.0 compiled successfully in 1091 ms
Done in 2.40s.
```

TBC -- MW, 07 January 2023

## Reference Documents

### Babel

- [Using Babel with TypeScript](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html)
- [Install Babel 7 and TypeScript](https://gist.github.com/rstacruz/648cb4dc68a76c761dc9e989832d9a50)
- [Setting up Babel and TypeScript](https://dev.to/benfixit/setting-up-babel-and-typescript-5ba8)

### Webpack

- [Webpack Configuration Languages](https://webpack.js.org/configuration/configuration-languages)
