# Typescript best practices

## Basic Webpack file for Node (Webpack version 4.x)
```
yarn add --dev webpack webpack-cli react babel-loader babel-plugin-module-resolver @babel/preset-env @babel/preset-react @babel/preset-typescript @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread 
```

```javascript
const path = require('path')
const serverConfig = {
  mode: 'production', // dev mode couses memory leak
  optimization: {
    minimize: false
  },
  devtool: 'source-map',
  resolve: {
    extensions: ['.ts', '.tsx'],
  },
  output: {
    libraryTarget: 'commonjs',
    path: path.join(__dirname, '.webpack'),
    filename: '[name].js',
  },
  target: 'node',
  module: {
    rules: [
      {
        test: /\.(ts)x?$/,
        loader: 'babel-loader',
        options: {
          presets: [
            ['@babel/preset-env', { targets: { node: '8.10' } }],
            '@babel/typescript',
            '@babel/preset-react'
          ],
          plugins: [
              '@babel/proposal-class-properties', // no need use bind(this) for methods in React class constructor
              '@babel/proposal-object-rest-spread', // easy immutable object: const a = { x: 1, ...b } // copy all attributes from b to a
              ["module-resolver", {
                "root": ["./src"],
                "alias": {
                  "shared": "./src/shared", // you can import files as shared/filename
                  "tests": "./src/tests",
                }
              }],
          ]
        }
      }
    ],
  },
  stats: 'minimal'
}

module.exports = config

```

With `@babel/proposal-class-properties` you can define state outside constructor:
```javascript
import * as React from 'react'

export class EmailButton extends React.Component<> {
  state = {
    hovered: false
  }
  public render () {
    // ...
  }
}

```

## Check Coding Style

use Tslint and Typescript compiler

```
yarn add --dev typescript tslint tslint-config-standard tslint-microsoft-contrib tslint-react
```

tslint.json
```json
{
  "defaultSeverity": "error",
  "extends": [
    "tslint:latest",
    "tslint-consistent-codestyle",
    "tslint-eslint-rules",
    "tslint-immutable",
    "tslint-microsoft-contrib",
    "tslint-config-prettier",
    "tslint-plugin-prettier"
  ],
  "jsRules": {},
  "rules": {
    "no-submodule-imports": [true, "src"],
    "no-implicit-dependencies": [true, "dev", ["src"]],
    "typedef": [false, "variable-declaration"], // Typedef enforcement for variables that contain a function => duplicate function declaration
    "no-use-before-declare": false, // useful only for var declaration
    "only-arrow-functions": true,
    "interface-name": [true, "never-prefix"],
    "jsx-boolean-value": ["never"],
    "prefer-template": false,
    "type-literal-delimiter": false,
    "newline-per-chained-call": false,
    "strict-boolean-expressions": false,
    "completed-docs": false,
    "mocha-no-side-effect-code": false,
    // Immutability rules
    "readonly-keyword": true,
    "no-let": true,
    "no-object-mutation": true,
    "no-delete": true,
    "prettier": [
      true,
      {
        "singleQuote":  true,
        "semi": false,
        "printWidth": 120
      }
    ]
  },
  "rulesDirectory": [
    "node_modules/tslint-eslint-rules/dist/rules",
    "node_modules/tslint-microsoft-contrib"
  ]
}


```

tsconfig.json
```json
{
  "compilerOptions": {
    "target": "es6",
    "module": "es6",
    "sourceMap": true,
    "strict": true,
    "moduleResolution": "node",
    "rootDir": "./src",
    "baseUrl": "./",
    "pretty": true,
    "declaration": true,
    "noImplicitAny": true,
    "noUnusedLocals": true,
    "removeComments": true,
    "noImplicitReturns": true,
    "suppressImplicitAnyIndexErrors": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": ".dist",
    "jsx": "react"
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

package.json
```json
{
  ...
  "scripts": {
    "lint": "tsc && tslint --project tsconfig.json --config tslint.json",
    "lint-fix": "prettier 'src/**/*' --write --single-quote --no-semi --print-width 120",
    ...
  }
}
```

## Errors

```
yarn add --dev extendable-error
```

Basicaly we have two types of the errors:

### Runtime error:
```javascript
import { ExtendableError } from 'extendable-error'

export class AppError extends ExtendableError {}
```

These errors we need catch:
```javascript
import { AppError } from './AppError'

try {
  // ...code throws AppError    
} catch (err) {
  if (err instanceof AppError) {
    // ...handle error
  }
  throw err
}
```

### Logic error:
```javascript
import { ExtendableError } from 'extendable-error'

export class AppLogicError extends ExtendableError {}
```
Do not catch these errors!

Typical use:
```javascript
const appSettings = loadAppSettings()
if (!appSettings.tableName) {
  throw new AppLogicError('Table name not defined.')
}
```

## Parsing JSON
```javascript
import { AppError } from './AppError'

export class InvalidJsonError extends AppError {
  constructor() {
    super('Invalid JSON.')
  }
}
```

```javascript
import { InvalidJsonError } from './InvalidJsonError'

export const parseJson = (data: string | undefined | null): any => {
  if (!data) {
    throw new InvalidJsonError()
  }
  try {
    return JSON.parse(data)
  } catch (e) {
    throw new InvalidJsonError()
  }
}
```
