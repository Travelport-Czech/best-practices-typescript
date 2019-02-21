# Typescript best practices

## Basic Webpack file for Node (Webpack version 4.x)
```
yarn add --dev webpack webpack-cli babel-loader babel-plugin-module-resolver @babel/preset-env @babel/preset-react @babel/preset-typescript @babel/plugin-proposal-class-properties @babel/plugin-proposal-object-rest-spread 
```

```javascript
const path = require('path')
const serverConfig = {
  mode: 'production', // dev mode couses memory leak
  optimization: {
    minimize: false
  },
  externals: [
    /aws-sdk/, // Available on AWS Lambda
  ],
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
    ordered: true
  }
  public render () {
    // ...
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
