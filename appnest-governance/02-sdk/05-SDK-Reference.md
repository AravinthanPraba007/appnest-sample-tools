# Appnest SDK Reference

Global reference for the Appnest custom SDK. Use for backend handlers: DB, HTTP, files, invoking other functions, and scheduled jobs. For usage rules see [04-AppNest-SDK-Usage-Rules.md](04-AppNest-SDK-Usage-Rules.md).

## Platform-provided dependency

**Do not add `@aravinthan_p/appnest-sdk-utils` (or `appnest-sdk-utils`) to `app-backend/package.json`.** The SDK is provided by the AppNest platform at runtime. Your backend code should `require('@aravinthan_p/appnest-sdk-utils')` (or the package name configured by the platform) without listing it as a dependency.

## Import

```javascript
const AppnestFunctions = require('@aravinthan_p/appnest-sdk-utils');
// or: require('appnest-sdk-utils');

const { $db, $file, $http, $next, $schedule, getTraceId } = AppnestFunctions;
const { ResultData } = require('@aravinthan_p/appnest-sdk-utils');  // for HTTP-style responses
```

## Quick reference

| Module       | Purpose                    | Main APIs / Methods |
|-------------|----------------------------|----------------------|
| **$db**     | Key/value storage          | `get`, `create`, `update`, `delete`; typed: `string`, `number`, `list`, `map`, `boolean` (each with get/create/update/delete and type-specific ops) |
| **$http**   | Outbound HTTP requests     | `request({ url, method, headers, body, query })` |
| **$file**   | File storage (signed URLs) | `getUploadUrl`, `getDownloadUrl`, `delete`, `list`, `exists` |
| **$next**   | Invoke another function    | `run({ functionName, payload, delay })` |
| **$schedule** | Scheduled jobs           | `create`, `get`, `update`, `pause`, `resume`, `delete` |
| **getTraceId** | Request trace ID        | `getTraceId()` → string (for logging/correlation) |

## $db

- **Key:** `key` (string, required, max 1000 chars).
- **Types:** `string`, `number`, `list`, `map`, `boolean`.
- **Common methods per type:** `get({ key })`, `create({ key, value })`, `update({ key, value })`, `delete({ key })`.
- **Extra:** `number.increment({ key })`, `number.decrement({ key })`; `list.append`, `list.prepend`, `list.updateItemAtIndex({ key, value, index })`, `list.removeItemAtIndex({ key, index })`.
- **Returns:** `Promise<{ data, status }>`.

## $http

- **Method:** `$http.request({ url, query, method, headers, body })`.
- **Params:** All required. `url` (string, https?://, max 2048); `method`: GET | POST | PUT | DELETE | PATCH; `headers`, `body`, `query`: objects with string values (max 1000 keys each, values max 1000 chars).
- **Returns:** `Promise<{ data, status }>`.

## $file

- **Methods:** `getUploadUrl`, `getDownloadUrl`, `delete`, `list`, `exists`.
- **Params:** `path` (string, required), `visibility` (optional, PUBLIC | PRIVATE, default PUBLIC).
- **Returns:** `Promise<{ data, status }>`.

## $next

- **Method:** `$next.run({ functionName, payload, delay })`.
- **Params:** `functionName` (string), `payload` (object), `delay` (number, seconds, default 0).
- **Returns:** `Promise<{ data, status }>`.

## $schedule

- **Methods:** `create`, `get`, `update`, `pause`, `resume`, `delete`.
- **Common:** `name` (string, max 30), `type` (ONE_TIME | CRON | RECURRING), `data` (object, at least one key).
- **ONE_TIME:** `runAt` (date string). **CRON:** `cronExpression`. **RECURRING:** `repeat: { frequency, timeUnit }` (timeUnit: MINUTES | HOURS | DAYS | WEEKS | MONTHS | YEARS).
- **Returns:** `Promise<{ data, status }>`.

## Usage in handlers

Use the SDK inside any function you export from `app-backend/server.js` (API or event handler). Handlers receive **`{ payload }`**:

```javascript
const { AppnestFunctions, ResultData } = require('@aravinthan_p/appnest-sdk-utils');
const { $db, $http, $next, getTraceId } = AppnestFunctions;

const myApiHandler = async ({ payload }) => {
  const traceId = getTraceId();

  await AppnestFunctions.$db.string.create({ key: 'my_key', value: payload.input });
  const row = await AppnestFunctions.$db.string.get({ key: 'config' });
  const res = await AppnestFunctions.$http.request({
    url: 'https://api.example.com',
    method: 'GET',
    headers: {},
    body: {},
    query: {},
  });
  await AppnestFunctions.$next.run({ functionName: 'otherFn', payload: { id: 1 }, delay: 0 });

  return new ResultData({ body: { row, res }, statusCode: 200 });
};

module.exports = { myApiHandler };
```

For full signatures and per-module docs, see the Appnest SDK package or platform docs.
