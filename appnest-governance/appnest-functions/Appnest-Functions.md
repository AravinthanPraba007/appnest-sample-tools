# Appnest Functions — Overview

Appnest separates “functions” into two layers: **backend helpers** used inside server handlers and a **frontend API** used to invoke those handlers by name.

## Backend AppnestFunctions
Use a **two-step** import from the same `require`: take **`AppnestFunctions`** and optionally **`ResultData`** from the package, then destructure **`$db`**, **`$fetch`**, etc. from **`AppnestFunctions`**. Do **not** do `const { $db } = require('@sparrowengg/appnest-app-sdk-utils')` — that is incorrect.

```js
const { AppnestFunctions, ResultData } = require('@sparrowengg/appnest-app-sdk-utils');
const { $db, $fetch, $file, $next, $schedule, getTraceId } = AppnestFunctions;
```

Do **not** pass **`$db`**, **`$fetch`**, **`$file`**, **`$next`**, **`$schedule`**, or **`getTraceId`** as parameters from one function to another (for example `doWork($fetch)`). Repeat the **`require`** + destructuring from **`AppnestFunctions`** at **module** scope in each backend file that needs them, and keep helper signatures limited to domain data. Full rationale and AI guidance: **[Backend-Appnest-Functions.md](Backend-Appnest-Functions.md)** (Import → *Helper usage (no DI-style parameters)*).

Available helpers (destructure only what you need):

- **`$db`** — typed key/value storage  
- **`$fetch`** — outbound HTTP requests  
- **`$file`** — signed upload/download and object operations  
- **`$next`** — invoke another backend function  
- **`$schedule`** — one-time, cron, or recurring jobs  
- **`getTraceId`** — request correlation  

The platform provides this package at runtime for backend code. Use it for storage, HTTP calls, file handling, chaining backend functions, and scheduling jobs.

**Return property names:** When you read a method’s result in handler code (including AI-generated code), use **only** the property names defined in **[Backend-Appnest-Functions.md](Backend-Appnest-Functions.md)** for that method—never substitute guessed names (`data`, `response`, `items`, etc.) when the reference specifies different keys or a direct value.

### Handler shape (API and events)

Each exported backend function is invoked with **one argument**: an object whose inputs live on **`payload`**. Write **`async ({ payload }) => { ... }`**, return a **plain object** or **`new ResultData({ body, statusCode })`**. **`ResultData`** comes from your app scaffold—import path depends on the template. Full contract and SDK reference: **[Backend-Appnest-Functions.md](Backend-Appnest-Functions.md)**.

## Frontend AppnestFunctions
From **`window.AppnestFunctions`**, take **`$app`** and **`await`** **`backend`**:

```js
const { $app } = window.AppnestFunctions;

const data = await $app.backend({
  functionName: 'myBackendHandler',
  functionPayload: { userId: 123 },
});
```

This can be used in the browser or app client. It triggers the platform’s backend invocation flow and returns the **parsed JSON response body** (not a `{ data, status }` wrapper). Use it whenever the UI needs to call a named server function.

The same **`window.AppnestFunctions`** object also includes **`$productParent`** (calls into the embedding product—name, region, and product-specific **`invoke`** actions) and **`$check`** (quick confirmation the bridge loaded). Product support for **`$productParent`** varies by host; see the frontend reference below.

For detailed information about function signatures, parameters, return shapes, and code-generation details, refer to:

- **[Backend-Appnest-Functions.md](Backend-Appnest-Functions.md)** — Backend `AppnestFunctions` API reference  
- **[Frontend-Appnest-Functions.md](Frontend-Appnest-Functions.md)** — Frontend `window.AppnestFunctions` reference