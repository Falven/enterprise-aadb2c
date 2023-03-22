# Configuration

Fran Aguilera
<fraguile@microsoft.com>

The following document contains recommended configuration, including samples, for an enterprise-grade AADB2C frontend application.

---

## Project Structure

This following is a recommended project structure:

```shell
PROJECT-NAME/                  # Root
├─ .vscode/                    # Tooling configuration
├─ templates/                  # Reusable HTML templates to inject
├─ node_modules/               # Npm packages
├─ out/                        # Static build output
├─ pages/                      # Routes correspond to site pages
│  ├─ home/                    # Home route
│  │  ├─ home.html             # Home route page
│  │  ├─ home.js               # Home route index js
│  ├─ .../                     # More page routes
│  ├─ index.js                 # Index JavaScript that loads JS for each page
├─ public/                     # Static-served files
│  ├─ images/                  # Static images
│  │  ├─ favicon.ico           # Site icon
│  │  ├─ web.config            # IIS webserver configuration
├─ styles/                     # SCSS/CSS Styles
│  │  ├─ variables.scss        # Global SCSS/CSS variables
│  │  ├─ globals.scss          # Global SCSS/CSS barrel file
│  │  ├─ my.module.scss        # SCSS/CSS Module
│  │  ├─ ...                   # More SCSS/CSS styles
├─ types/                      # Typescript types (If using TS)
├─ utils/                      # TS/JS Utilities
├─ .env                        # Environment files that overwrite all
├─ .env.development            # Environment files for development environment
├─ .env.local                  # Environment files for local development environment
├─ .env.production             # Environment files for production environment
├─ .eslintignore               # ESLint-ignored files
├─ .eslintrc.json              # ESLint configuration
├─ .gitignore                  # git-ignored files
├─ .npmrc                      # npm configuration
├─ .prettierignore             # Prettier-ignored files
├─ .stylelintignore            # Stylelint-ignored files
├─ .stylelintrc.json           # Stylelint configuration
├─ package.json                # npm packages, scripts and module information
├─ pnpm-lock.yaml              # pnpm lock file
├─ postcss.config.js           # PostCSS configuration
├─ README.md                   # 😃
├─ tsconfig.json               # Typescript configuration
├─ webpack.config.js           # Webpack configuration
```

---

## Recommended Package Manager:

[pnpm](https://pnpm.io/)

---

## Recommended Package Scripts:

`package.json`

```json
{
  "scripts": {
    "build": "webpack",
    "build-dev": "webpack build --devtool source-map --mode=development",
    "watch": "webpack --watch",
    "watch-dev": "webpack --watch --devtool source-map --mode=development",
    "lint": "pnpm run lint:js && pnpm run lint:css && pnpm run lint:json",
    "lint:fix": "pnpm run lint:js:fix && pnpm run lint:css:fix && pnpm run lint:json:fix",
    "lint:css": "stylelint **/*.{scss,css}",
    "lint:css:fix": "stylelint --fix **/*.{scss,css}",
    "lint:js": "eslint \"**/*.js\"",
    "lint:js:fix": "pnpm run lint:js --fix",
    "lint:json": "prettier --check \"./**/*.json\"",
    "lint:json:fix": "prettier --write \"./**/*.json\""
  }
}
```

---

## Recommended Tooling

### VSCode

Recommended Extensions:

`.vscode/extensions.json`

```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "stylelint.vscode-stylelint",
    "mrmlnc.vscode-scss",
    "redhat.vscode-xml"
  ]
}
```

Recommended launch configurations:

Useful for debugging webpack configurations.

`.vscode/launch.json`

```json
{
  [
    {
      "name": "Debug Webpack",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/webpack/bin/webpack.js",
      "args": ["--config", "${workspaceFolder}/webpack.config.js"],
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

Recommended tasks:

`.vscode/tasks.json`

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start IIS",
      "command": "iisexpress.exe",
      "args": ["/path:${workspaceFolder}\\out", "/port:8016"],
      "type": "process",
      "options": {
        "cwd": "C:\\Program Files\\IIS Express"
      }
    },
    {
      "label": "Dev: Start IIS and Watch",
      "dependsOn": ["pnpm run watch-dev", "Start IIS"],
      "options": {
        "cwd": "${workspaceFolder}"
      }
    }
  ]
}
```

Recommended workspace settings:

`.vscode/settings.json`

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "css.validate": false,
  "less.validate": false,
  "scss.validate": false,
  "stylelint.enable": true,
  "stylelint.configFile": ".stylelintrc.json",
  "stylelint.validate": ["scss", "sass", "css"],
  "stylelint.packageManager": "pnpm",
  "stylelint.snippet": ["scss", "sass", "css"],
  "[xml]": {
    "editor.defaultFormatter": "redhat.vscode-xml"
  }
}
```

---

## Recommended Bundler

### Webpack

There are a number of plugins to set up that support the different features required by our application:

[Read here](https://handlebarsjs.com/installation/integrations.html) on integrating [the handlebars-webpack-plugin](https://github.com/sagold/handlebars-webpack-plugin) for templating support.

[Read here](https://www.npmjs.com/package/html-webpack-plugin) on integrating [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin) for html support.

[Read here](https://github.com/webpack-contrib/mini-css-extract-plugin#getting-started) on integrating [mini-css-extract-plugin](https://www.npmjs.com/package/mini-css-extract-plugin) for css module support.

[Read here](https://github.com/mrsteele/dotenv-webpack#usage) on integrating [dotenv-webpack](https://www.npmjs.com/package/dotenv-webpack) for environment variable injection.

[Read here](https://github.com/webpack-contrib/copy-webpack-plugin#getting-started) on integrating [copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) to copy assets to `out/`.

Barebones webpack configuration:

```js
const path = require('path');

module.exports = {
  entry: './index.js',
  target: ['web', 'es2022'],
  module: {
    // ...
  },
  output: {
    filename: 'index.js',
    path: path.resolve(__dirname, 'out'),
    library: {
      type: 'module',
    },
    publicPath: `${process.env.AZURE_STORAGE_ACCOUNT_URL}/`,
  },
  experiments: {
    outputModule: true,
  },
  plugins: [
    // ...
  ],
};
```

---

## Recommended formatter

### Prettier

Recommended `prettierrc.json`

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "consistent",
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "proseWrap": "never",
  "htmlWhitespaceSensitivity": "strict",
  "endOfLine": "lf",
  "overrides": [
    {
      "files": "*.json",
      "options": {
        "parser": "json"
      }
    },
    {
      "files": "*.yaml",
      "options": {
        "parser": "yaml"
      }
    }
  ]
}
```

Recommended `.prettierignore`:

```
node_modules/
out/
pnpm-lock.yaml
```

---

## Recommended JS Linting

### ESLint

Recommended `.eslintignore`:

```
**/*.*
!pages/**/*.ts
!pages/**/*.js
!types/**/*.ts
!types/**/*.js
!utils/**/*.ts
!utils/**/*.js
```

Recommended `.eslintrc.json`:

```json
{
  "plugins": ["@typescript-eslint"],
  "extends": ["plugin:@typescript-eslint/recommended", "prettier"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```

---

## Recommended Style Linting

### Stylelint

Recommended `.stylelintignore`:

```
**/*.*
!styles/**/*.scss
!styles/**/*.sass
!styles/**/*.css
```

Recommended `.stylelintrc.json` if using scss:

```json
{
  "extends": ["stylelint-config-standard-scss", "stylelint-config-css-modules"]
}
```

Recommended `.stylelintrc.json` if using css:

```json
{
  "extends": ["stylelint-config-standard", "stylelint-config-css-modules"]
}
```

---

## PurgeCSS

While I haven't tested it in an AADB2C application, PurgeCSS can be used to purge unwanted CSS and reduce the bundle size.

Here's a barebones configuration:

```js
const purgecss = [
  '@fullhuman/postcss-purgecss',
  {
    content: ['./pages/**/*.{ts,js}', './utils/**/*.{ts,js}'],
    css: ['./styles/**/*.{scss,sass,css}'],
    defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
    safelist: ['html', 'body'],
  },
];
```

---

## Git

Recommended `.gitignore`:

```
# dependencies
/node_modules

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# local env files
.env*.local

# typescript
*.tsbuildinfo

# Ignore all starting with `.`, except `.vscode`
.*/
!.vscode/
```
