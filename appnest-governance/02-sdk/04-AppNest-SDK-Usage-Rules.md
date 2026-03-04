# Appnest SDK Usage Rules

All external operations must use Appnest SDK primitives.

**Package dependency:** Do not add `@aravinthan_p/appnest-sdk-utils` to `app-backend/package.json`. The SDK is provided by the platform at runtime.

## 1. $http

All external HTTP calls MUST use $http.

Direct use of axios/fetch is prohibited.

## 2. $db

All persistent state MUST use $db.

In-memory storage is not allowed for cross-invocation state.

## 3. $schedule

Long-running or chained execution MUST use $schedule.

Direct recursion without scheduling is not allowed.

## 4. $file

File uploads and processing MUST use $file.

Files must be validated before processing.

## 5. ResultData

Handlers should return **ResultData** for explicit status and body control (e.g. non-200, error payloads). Plain objects are acceptable for simple success responses. Throwing raw errors without a structured return is prohibited.