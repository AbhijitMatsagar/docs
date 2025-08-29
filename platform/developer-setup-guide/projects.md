# Projects

`POST /api/v1/portal/project`

***

## Overview

This API allows users to manage venues and bookings.

### Flow

1. Authenticate with the `/auth/login` endpoint.
2. Fetch available venues using `/venues`.
3. Book a venue with `/bookings`.
4. Track your bookings using `/bookings/{id}`.

***

## 🔹 Performance (JMeter Test Results)

| Metric       | Value    |
| ------------ | -------- |
| Samples      | 1000     |
| Avg. Latency | 9042 ms  |
| Min          | 133 ms   |
| Max          | 36246 ms |
| Std. Dev.    | 9891.37  |
| Error %      | 35.40%   |
| Throughput   | 17.8/sec |

***

## 🔹 Header Parameters

| Name | Type   | Example  |
| ---- | ------ | -------- |
| at   | string | `{{at}}` |

***

## 🔹 Request Body

```json
{
  "name": "projcv vvt test 1",
  "description": "test project description",
  "tags": ["test1"]
}
```

***

## 🔹 Code Samples

```javascript
const response = await fetch("https://app-dev.spatiosynth.com/api/v1/portal/project", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    "name": "projcv vvt test 1",
    "description": "test project description",
    "tags": ["test1"]
  })
});

const data = await response.json();
```

***

## 🔹 Responses

<details>

<summary>✅ 200 OK</summary>

```json
{
  "id": 6,
  "name": "project test 5",
  "description": "test project description 3",
  "projectUserId": 1,
  "deleted": false,
  "createdAt": "2024-09-03T15:29:44.666Z",
  "updatedAt": "2024-09-03T15:29:44.666Z",
  "projectTags": [
    { "id": 4, "projectId": 6, "tag": "ptag5" },
    { "id": 5, "projectId": 6, "tag": "ptag3" }
  ]
}
```

</details>

<details>

<summary>❌ 400 Bad Request</summary>

```json
{ "message": "Invalid request" }
```

</details>
