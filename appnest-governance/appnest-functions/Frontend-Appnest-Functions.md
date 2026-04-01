# `window.AppnestFunctions` (developer reference)

**What it is:** A global object added when **`client.js`** runs in your marketplace app iframe. Use it from your frontend to call the **Sparrow Apps backend** from supported APIs below.

**Before you call it:** Load `client.js` first. Your app should run inside the product iframe. For **`$app.backend`**, the app URL should include **`appId`** and **`appVersionId`** query params.

---

## Exposed API

```js
window.AppnestFunctions = {
  $check,
  $app: { backend },
};
```

### `$check()`

Confirms the bridge loaded. Returns synchronously.

```js
const { data } = window.AppnestFunctions.$check();
// data.message — e.g. greeting from the client bridge
```

### `$app.backend({ functionName, functionPayload?, options? })`

Calls your server-side handler via the platform (`serverMethod` + payload). Returns a **`Promise`** that resolves to the API JSON body. Optional `options.timeout` (default ~120s).

```js
const result = await window.AppnestFunctions.$app.backend({
  functionName: 'myServerHandler',
  functionPayload: { id: 1 },
});
```

---

## Object shape (JavaScript / JSDoc)

Paste near your app code if your editor uses JSDoc for hints. `window.AppnestFunctions` is still provided by **`client.js`** — this only documents the shape.

```js
/**
 * Shape of `window.AppnestFunctions` (injected by client.js).
 *
 * @typedef {object} AppnestFunctions
 * @property {() => { data: { message: string } }} $check
 * @property {{ backend: AppBackendInvoke }} $app
 */

/**
 * @callback AppBackendInvoke
 * @param {object} args
 * @param {string} args.functionName
 * @param {*} [args.functionPayload]
 * @param {{ timeout?: number }} [args.options]
 * @returns {Promise<*>}
 */
```

On HTTP errors (e.g. 401 or status > 300) backend calls may show a toastr and reject; timeouts reject with an `Error`.
