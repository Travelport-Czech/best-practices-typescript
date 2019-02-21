# Typescript best practices

## Errors
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
