# React App From Scratch

Setting up a Web App using TypeScript, React, Jest, Babel, Webpack, Babel, ESLint, Prettier, and yarn (classic).

**NOT** going to use `create-react-app` (CRA).

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

Make sure that `node` is installed on the system. In addition, install `yarn` if it's not already present on your system:

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

Let's start with TypeScript. Since, at build time, all the TypeScript code is transpiled to JavaScript, we will add TypeScript in the `devDependencies` only. Let's also add the typings for the Node libraries.

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

When all goes well, we should see the `build` folder as specified in `tsconfig.json` under `outDir`. If you didn't specify the `outDir` parameter, the resulting `index.js` will be written to `src` instead.

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
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
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

Let's add Babel to the mix. The idea is that Babel will be responsible for transpiling the TypeScript code to JavaScript and `tsc` will be responsible for type-checking.

```
yarn add --dev @babel/core @babel/cli @babel/preset-env @babel/preset-typescript
```

Create the file `babel.config.json` with the following contents:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"]
}
```

We'll also have to adapt `tsconfig.json` since we need to make sure that `tsc` will not be creating `*.js` files any longer. Add the following lines to the `compilerOptions` section in `tsconfig.json`.

```json
{
  ...
  /* Don't emit anything; allow Babel to transpile. */
  "noEmit": true,
  /* Ensure that Babel can safely transpile files in the TypeScript project */
  "isolatedModules": true
  ...
}
```

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

```shell
> yarn run build
yarn run v1.22.19
$ babel src --out-dir build --extensions .ts
Successfully compiled 1 file with Babel (582ms).
Done in 1.27s.
```

As can be seen, the build now utilises Babel.

We should also check that running `tsc` really doesn't emit anything.

```shell
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

Oops! That's apparently a side effect of using `isolatedModules`. Let's do as the message suggests and change `index.ts` to include an empty `export` statement:

```typescript
console.log("Hello, world!");

export {};
```

Now, let's run `tsc` again:

```shell
> yarn run tsc
yarn run v1.22.19
$ tsc
Done in 2.91s.
```

Perfect! 

If you did actually delete the build folder -- just to check, of course -- you will see now that running `tsc` didn't actually create anything. It is now also safe to remove the following line from `tsconfig.json`:

```json
"outDir": "build"
```

TBC -- MW, 06 January 2023

## Reference Documents

### Babel

- [Using Babel with TypeScript](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html)
- [Install Babel 7 and TypeScript](https://gist.github.com/rstacruz/648cb4dc68a76c761dc9e989832d9a50)
- [Setting up Babel and TypeScript](https://dev.to/benfixit/setting-up-babel-and-typescript-5ba8)
