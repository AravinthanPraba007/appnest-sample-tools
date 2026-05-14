# Backend Appnest Functions — Developer Reference

This document is the **single source of truth** for the Appnest custom SDK functions. Use it for **code generation**, auto-completion, and integration in your app.

**For AI code generation:** (1) This file is self-contained; all SDK function signatures, parameters, return shapes, and constraints are below, plus the **backend handler contract** (single argument object with **`payload`**, return plain object or **`new ResultData({ body, statusCode })`**). (2) **Import pattern:** Take **`AppnestFunctions`** and **`ResultData`** from the runtime package your scaffold uses (often `@sparrowengg/appnest-app-sdk-utils`): `const { AppnestFunctions, ResultData } = require('@sparrowengg/appnest-app-sdk-utils'); const { $db, $fetch, $file, $next, $schedule, getTraceId } = AppnestFunctions;` (destructure only the modules you need from `AppnestFunctions`). Do not use `const { $db } = require('@sparrowengg/appnest-app-sdk-utils')` — that is incorrect. If your template uses a different `require` path, follow it; **`ResultData`** comes from your app scaffold—import path depends on the template (it is usually exported beside **`AppnestFunctions`**). (3) **Return field names:** When consuming SDK results, use **only** the property names documented in this file for each method (for example `$fetch.request` → `headers`, `body`, `status`; `$file.list` → `keys`). Do **not** guess aliases such as `data`, `response`, `items`, or `objects` when this document specifies different keys—wrong names fail at runtime. (4) **SDK helpers are not injectable arguments:** Do **not** pass **`$db`**, **`$fetch`**, **`$file`**, **`$next`**, **`$schedule`**, or **`getTraceId`** as function parameters (for example `helper($fetch)`). Always obtain them by destructuring from **`AppnestFunctions`** at **module** scope per the [Import](#import) section and **Helper usage (no DI-style parameters)** below; pass only domain data between functions.

**Do not add `@sparrowengg/appnest-app-sdk-utils` (or `appnest-app-sdk-utils`) to `app-backend/package.json`.** The SDK is provided by the AppNest platform at runtime. Your backend code should `require('@sparrowengg/appnest-app-sdk-utils')` (or the package name configured by the platform) without listing it as a dependency.

---

## Import

```javascript
const { AppnestFunctions, ResultData } = require('@sparrowengg/appnest-app-sdk-utils');
// or from path per your app scaffold:
// const { AppnestFunctions, ResultData } = require('<path-to-sdk>/appnestFunctions');

const { $db, $file, $fetch, $next, $schedule, getTraceId } = AppnestFunctions;
```

### Helper usage (no DI-style parameters)

Do **not** thread **`$db`**, **`$fetch`**, **`$file`**, **`$next`**, **`$schedule`**, or **`getTraceId`** through function signatures as parameters (for example `const callRemote = ($fetch) => { ... }` then `callRemote($fetch)`). That pattern obscures where platform APIs come from, is easy to wire wrong, and reads like generic dependency injection, which Appnest does not require.

In **each** backend file that needs SDK I/O, use the **same two-step import** as above: `require` **`AppnestFunctions`**, then **`const { $fetch, ... } = AppnestFunctions`** at **module** scope. Plain helpers and shared utilities should accept normal values (`payload`, ids, config)—**not** **`$*`** helpers—so they close over **`$fetch`** et al. from the file’s top-level destructuring or call siblings in the same module.

---

## Backend handler contract

- **Function signature:** Each backend handler is invoked with a single argument object; destructure **`payload`** from it—**`({ payload })`**. Return a plain object, or **`new ResultData({ body, statusCode })`** when you want to set HTTP-style status and response body explicitly (**`ResultData`** comes from your app scaffold—import path depends on the template).

```javascript
const { AppnestFunctions, ResultData } = require('@sparrowengg/appnest-app-sdk-utils');
const { $db, $fetch, $file, $next, $schedule, getTraceId } = AppnestFunctions;

const function1 = async ({ payload }) => {
  console.log("function1 is invoked");
  const userName = payload.userName;
  return {
    "key1": "value1",
    "key2": userName,
  };
};

const function2 = async ({ payload }) => {
  console.log("function2 is invoked");
  const userName = payload.userName;
  return new ResultData({
    body: {
      message: 'Function 2 executed successfully..!!@@@@!!.',
      userName,
    },
    statusCode: 200,
  });
};
```

Export these from **`app-backend/server.js`** and declare each name under **`backend_api_functions`** and/or **`event_listener_functions`** as applicable. Inside handlers, use **`getTraceId()`** and the **`$db` / `$fetch` / `$file` / `$next` / `$schedule`** APIs documented below.

---

## Quick reference

| Module      | Purpose                    | Main APIs / Methods |
|------------|----------------------------|----------------------|
| **$db**    | Key/value storage          | `get`, `set`, `delete`; typed: `string` / `number` (get/set/delete + number increment/decrement), `list` (get/set/append/prepend/updateItemAtIndex/removeItemAtIndex), `map` (get/set/delete), `boolean` (get/set/delete) |
| **$fetch** | Outbound HTTP requests     | `request({ url, method, headers, body, options? })` |
| **$file**  | File storage (signed URLs) | `getUploadUrl`, `getDownloadUrl`, `delete`, `list`, `exists` |
| **$next**  | Invoke another function    | `run({ functionName, functionPayload, delay })` |
| **$schedule** | Scheduled jobs          | `create`, `get`, `update`, `pause`, `resume`, `delete` |
| **getTraceId** | Request trace ID       | `getTraceId()` → string (for logging/correlation) |

---

## Module summary

Brief overview of each module.

| Module      | Summary |
|-------------|--------|
| **$db**     | Key/value by type: STRING, NUMBER, LIST, MAP, BOOLEAN. Common param: `key` (string). |
| **$fetch**  | `$fetch.request({ url, method, headers, body, options? })`. **options:** `isOauth` (boolean, default false), `maxAttempts` (number, default 0). Methods: GET, POST, PUT, DELETE, PATCH. Use `<%=installation_parameters.<key>%>` for manifest params; use `<%=user_oauth.access_token%>` in headers for platform access_token; framework replaces at runtime. |
| **$file**   | `getUploadUrl`, `getDownloadUrl`, `delete`, `list`, `exists`. Params: `path`, `visibility` (optional). |
| **$next**   | `$next.run({ functionName, functionPayload, delay })`. Delay in seconds. |
| **$schedule** | Job kinds: ONE_TIME, CRON, RECURRING. `jobName`, `jobType`, `jobPayload`, `jobConfig` (create/update); `jobConfig` holds `runAt` / `cronExpression` / `repeat` by type. |

---

## Code reference — function signatures & param details

**Using `$` function return values (human + AI code generation):**

- Use the **exact** response property names from the tables and summaries below for each call (`$db`, `$fetch`, `$file`, `$next`, `$schedule`).
- Do **not** substitute invented or “conventional” names (e.g. `data`, `result`, `records`, `items`) where this file defines a different shape.

**Return shapes (summary):**  
- **$db:** Get methods return the value directly (string, number, array, object, boolean). Create/update/delete return nothing (`Promise<void>`).  
- **$fetch:** `request` returns `Promise<{ headers, body, status }>`.  
- **$file:** All methods return the response object directly (e.g. `{ preSignedUrl }`, `{ deleted }`, `{ keys }`, `{ exists }`). No `{ data, status }` wrapper.  
- **$next:** `run` returns the API response body directly (e.g. `Promise<{ success: boolean }>`).  
- **$schedule:** All methods return nothing (`Promise<void>`).

---

### $db

**Common param for all $db methods:** `key` (string, required, non-empty, max 1000 chars).

**Return shape:** **Get** methods resolve to the stored value (`Promise<string>`, `Promise<number>`, `Promise<unknown[]>` for lists, `Promise<object>` for maps, `Promise<boolean>` for booleans). **Set**, **delete**, **increment**, **decrement**, **append**, **prepend**, **updateItemAtIndex**, and **removeItemAtIndex** return **`Promise<void>`** (no return payload to read).

| Function | Params | Types / constraints | Return |
|----------|--------|----------------------|--------|
| `$db.get({ key })` | key | string | `Promise<string>` |
| `$db.set({ key, value })` | key, value | key: string; value: string (writes string at key; same backend op as `$db.string.set`) | `Promise<void>` |
| `$db.delete({ key })` | key | string | `Promise<void>` |
| `$db.string.get({ key })` | key | string | `Promise<string>` |
| `$db.string.set({ key, value })` | key, value | key: string; value: string | `Promise<void>` |
| `$db.string.delete({ key })` | key | string | `Promise<void>` |
| `$db.number.get({ key })` | key | string | `Promise<number>` |
| `$db.number.set({ key, value })` | key, value | key: string; value: number | `Promise<void>` |
| `$db.number.delete({ key })` | key | string | `Promise<void>` |
| `$db.number.increment({ key, value })` | key, value | key: string; value: number (optional, for increment amount) | `Promise<void>` |
| `$db.number.decrement({ key, value })` | key, value | key: string; value: number (optional, for decrement amount) | `Promise<void>` |
| `$db.list.get({ key })` | key | string | `Promise<unknown[]>` (array of stored items) |
| `$db.list.set({ key, value })` | key, value | key: string; value: array | `Promise<void>` |
| `$db.list.append({ key, value })` | key, value | key: string; value: array | `Promise<void>` |
| `$db.list.prepend({ key, value })` | key, value | key: string; value: array | `Promise<void>` |
| `$db.list.updateItemAtIndex({ key, value, index })` | key, value, index | key: string; value: array; index: number | `Promise<void>` |
| `$db.list.removeItemAtIndex({ key, index })` | key, index | key: string; index: number | `Promise<void>` |
| `$db.map.get({ key })` | key | string | `Promise<object>` (JSON object, non-array) |
| `$db.map.set({ key, value })` | key, value | key: string; value: object (JSON-serializable) | `Promise<void>` |
| `$db.map.delete({ key })` | key | string | `Promise<void>` |
| `$db.boolean.get({ key })` | key | string | `Promise<boolean>` |
| `$db.boolean.set({ key, value })` | key, value | key: string; value: boolean | `Promise<void>` |
| `$db.boolean.delete({ key })` | key | string | `Promise<void>` |

---

### $fetch

| Function | Params | Types / constraints |
|----------|--------|----------------------|
| `$fetch.request({ url, method, headers, body, options? })` | url, method, headers, body, options | **url** (string, required). **method** (string, required): `GET` \| `POST` \| `PUT` \| `DELETE` \| `PATCH`. **headers** / **body** (object, optional, default `{}`). **options** (object, optional): **isOauth** (boolean, default false), **maxAttempts** (number, default 0). **Return:** `Promise<{ headers, body, status }>`. **Runtime replacement:** Use `<%=installation_parameters.<key>%>` and `<%=user_oauth.access_token%>` in headers. See [fetch.md](fetchFunction/fetch.md). |

---

### $file

**Common params:** `path` (string, required), `visibility` (string, optional, default `"PUBLIC"`: `"PUBLIC"` \| `"PRIVATE"`). All return the response object directly (no `{ data, status }` wrapper).

| Function | Params | Return |
|----------|--------|--------|
| `$file.getUploadUrl({ path, visibility })` | path, visibility | `Promise<{ preSignedUrl: string }>` |
| `$file.getDownloadUrl({ path, visibility })` | path, visibility | `Promise<{ preSignedUrl: string }>` |
| `$file.delete({ path, visibility })` | path, visibility | `Promise<{ deleted: boolean }>` |
| `$file.list({ path, visibility })` | path, visibility | `Promise<{ keys: string[] }>` |
| `$file.exists({ path, visibility })` | path, visibility | `Promise<{ exists: boolean }>` |

---

### $next

| Function | Params | Types / constraints | Return |
|----------|--------|----------------------|--------|
| `$next.run({ functionName, functionPayload, delay })` | functionName, functionPayload, delay | **functionName** (string, required). **functionPayload** (object, required). **delay** (number, optional, default 0): seconds, >= 0. | `Promise<object>` (API response body, e.g. `{ success: boolean }`) |

---

### $schedule

**Common for create/update:** `jobName` (string, required, max 30), `jobType` (string, required: `ONE_TIME` \| `CRON` \| `RECURRING`), `jobPayload` (object, required, at least one key), `jobConfig` (object, required: type-specific fields only). All schedule methods return `Promise<void>`. (The HTTP layer still sends `name`, `type`, `data`, and top-level `runAt` / `cronExpression` / `repeat`; the SDK maps from `jobName`, `jobType`, `jobPayload`, and expands `jobConfig`. For `RECURRING`, `repeat` is sent as `{ every, unit }`.)

| Function | Params | Types / constraints | Return |
|----------|--------|----------------------|--------|
| `$schedule.create({ jobName, jobType, jobPayload, jobConfig })` | jobName, jobType, jobPayload, jobConfig | **jobConfig for ONE_TIME:** `{ runAt }` (string, required). **CRON:** `{ cronExpression }` (string, required). **RECURRING:** `{ repeat: { frequency, timeUnit } }` — frequency (integer >= 1; >= 10 if timeUnit is MINUTES), timeUnit: `MINUTES` \| `HOURS` \| `DAYS` \| `WEEKS` \| `MONTHS` \| `YEARS`. | `Promise<void>` |
| `$schedule.get({ jobName, jobType })` | jobName, jobType | jobName: string; jobType: ONE_TIME \| CRON \| RECURRING | `Promise<void>` |
| `$schedule.update({ jobName, jobType, jobPayload, jobConfig })` | same as create | Same `jobConfig` rules as create. | `Promise<void>` |
| `$schedule.pause({ jobName, jobType })` | jobName, jobType | string, string | `Promise<void>` |
| `$schedule.resume({ jobName, jobType })` | jobName, jobType | string, string | `Promise<void>` |
| `$schedule.delete({ jobName, jobType })` | jobName, jobType | string, string | `Promise<void>` |

---

### getTraceId

| Function | Params | Return |
|----------|--------|--------|
| `getTraceId()` | none | string (trace ID for logging/correlation) |

---

## Usage in handlers

Typical pattern combining the **handler contract** with AppnestFunctions:

```javascript
const { AppnestFunctions, ResultData } = require('@sparrowengg/appnest-app-sdk-utils');
const { $db, $fetch, $file, $next, $schedule, getTraceId } = AppnestFunctions;

const myHandler = async ({ payload }) => {
  const traceId = getTraceId();
  const row = await $db.string.get({ key: "config" }); // row is the string value
  const res = await $fetch.request({
    url: "https://api.example.com",
    method: "GET",
    headers: {},
    body: {},
  }); // optional: options: { isOauth, maxAttempts }
  const nextResult = await $next.run({
    functionName: "otherFn",
    functionPayload: { id: 1 },
    delay: 0,
  }); // nextResult is the API response body
  return { traceId, row, externalStatus: res.status };
};
```

**Code generation:** Use this single file as the only input; it contains the handler contract (`({ payload })`, plain object vs `new ResultData`) and all SDK function names, signatures, param details, and **return property names** needed to generate correct calls to `$db`, `$fetch`, `$file`, `$next`, `$schedule`, and `getTraceId`—including how handlers read each method’s result (no guessed field names). Do **not** generate code that passes **`$*`** helpers or **`getTraceId`** as arguments between functions; always use the **Import** pattern at module scope.
