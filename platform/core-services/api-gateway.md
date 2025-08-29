# API Gateway

Here’s a **structured, refined, and GitBook-friendly version** of your API Gateway documentation. I've formatted it cleanly using headings, tables, code blocks, and bullet points for clarity and easy navigation in GitBook.

***

## 📘 API Gateway – NGINX + Lua + Docker

***

### 1. What Is the API Gateway?

The **API Gateway** is the single entry point for all external HTTP traffic into our backend system. Built with **NGINX + Lua** and deployed via **Docker**, it acts as a **reverse proxy**, routing client requests to microservices based on URL paths.

#### Key Responsibilities:

* Acts as a reverse proxy
* Performs request routing
* Handles authentication and authorization via Lua
* Validates tokens by communicating with the IAM service
* Returns unified error responses in JSON format

***

### 2. Why We Use an API Gateway

#### Without a Gateway:

* Each client must know every microservice's address.
* Each microservice must implement its own auth, logging, and rate-limiting.
* Error handling and security logic is fragmented.

#### With Our Gateway:

| Feature            | Benefit                         |
| ------------------ | ------------------------------- |
| Single entry point | `http://localhost/api/v1/*`     |
| Centralized auth   | Lua scripts + IAM service       |
| Unified responses  | JSON error format               |
| Dockerized         | Easy to deploy and scale        |
| Internal routing   | NGINX upstreams handle proxying |

***

### 3. Technology Stack

| Component          | Role                                |
| ------------------ | ----------------------------------- |
| **NGINX**          | Reverse proxy, Lua execution engine |
| **OpenResty**      | Lua-enabled NGINX distribution      |
| **Lua**            | Handles token-based auth logic      |
| **Docker Compose** | Container orchestration             |
| **PostgreSQL**     | Backend database (used by services) |

***

### 4. Core Functionalities

* Routes `/api/v1/portal` to `portal-api`
* Routes `/api/v1/ml` to `ml-api`
* Routes `/api/v1/iam` to `iam-api-gateway`

#### Lua Script Execution

| Route     | Lua Script        |
| --------- | ----------------- |
| `/portal` | `portal_auth.lua` |
| `/ml`     | `ml_auth.lua`     |

#### Authorization

* Lua scripts internally call:
  * `/portal-authorize`
  * `/ml-authorize`

#### Additional Features:

* Custom error responses (400–404)
* Adds CORS & required headers
* Handles preflight requests (OPTIONS)

***

### 5. Key File Overview

| File Path             | Purpose                       |
| --------------------- | ----------------------------- |
| `nginx.conf`          | Main NGINX configuration      |
| `servers.conf`        | Upstream service mapping      |
| `api_conf.d/api.conf` | Routing + Lua auth logic      |
| `json_errors.conf`    | JSON-based error pages        |
| `lua/*.lua`           | Lua authorization logic       |
| `Dockerfile`          | Gateway container build logic |

***

### 6. nginx.conf – Main NGINX Config

#### Highlights

* Loads `servers.conf` and `api_conf.d/*.conf`
* Handles large timeouts (`3000s`)
* Restricts request body size (10MB)
* Optional: Enable `json_errors.conf`

```nginx
include /etc/nginx/servers.conf;
include /etc/nginx/api_conf.d/*.conf;
proxy_read_timeout 3000s;
proxy_connect_timeout 3000s;
client_max_body_size 10M;
```

> 📌 Test config:\
> `openresty -t -c /etc/nginx/conf.d/nginx.conf`

***

### 7. servers.conf – Upstream Definitions

#### Example

```nginx
upstream portal-api {
  server portal-api:3000;
}

upstream ml-api {
  server ml-api:3000;
}

upstream iam-api-gateway {
  server iam-api-gateway:80;
}
```

#### Benefits:

* Easy to change ports/services without updating `api.conf`
* Load balancing support via multiple `server` entries
* Environment-agnostic naming using Docker service names

***

### 8. api.conf – Main API Routing

| Route            | Action                                           |
| ---------------- | ------------------------------------------------ |
| `/api/v1/portal` | Run `portal_auth.lua` then proxy to `portal-api` |
| `/api/v1/ml`     | Run `ml_auth.lua` then proxy to `ml-api`         |
| `/api/v1/iam`    | Direct proxy to IAM (no Lua check)               |

#### Internal Auth Subroutes:

```nginx
location = /portal-authorize {
  internal;
  proxy_pass http://iam-api-gateway/api/v1/iam/portal/authenticate;
  proxy_pass_request_headers on;
}
```

***

### 9. Lua Auth Scripts – `portal_auth.lua` & `ml_auth.lua`

Both Lua files follow the same pattern:

```lua
local res = ngx.location.capture("/portal-authorize")

if res.status == 200 then
    ngx.exit(ngx.OK)
elseif res.status == 204 then
    -- CORS preflight
    ngx.header["Access-Control-Allow-Origin"] = "*"
    ngx.exit(204)
else
    ngx.status = res.status
    ngx.say(res.body)
    ngx.exit(res.status)
end
```

#### Behavior Summary:

| Status  | Meaning                           |
| ------- | --------------------------------- |
| 200     | Auth success → forward to backend |
| 204     | CORS preflight                    |
| 401/403 | Unauthorized → return IAM error   |

***

### 10. json\_errors.conf – Custom Error Pages

Enables JSON error formatting for:

| Code | Response                                  |
| ---- | ----------------------------------------- |
| 400  | `{"status":400,"message":"Bad request"}`  |
| 401  | `{"status":401,"message":"Unauthorized"}` |
| 403  | `{"status":403,"message":"Forbidden"}`    |
| 404  | `{"status":404,"message":"Not found"}`    |

#### Setup:

```nginx
proxy_intercept_errors on;
default_type application/json;
include /etc/nginx/json_errors.conf;
```

***

### 11. Dockerfile – Gateway Container Setup

#### Base Image:

```dockerfile
FROM openresty/openresty:1.19.9.1-5-alpine-fat
```

#### Key COPY Instructions:

```dockerfile
COPY ./nginx.conf /etc/nginx/conf.d/nginx.conf
COPY ./servers.conf /etc/nginx/servers.conf
COPY ./api_conf.d /etc/nginx/api_conf.d
COPY ./json_errors.conf /etc/nginx/json_errors.conf
COPY lua /usr/local/openresty/nginx/lua
COPY html /usr/share/nginx/html
```

#### Permissions:

```dockerfile
RUN chmod -R a+rX /usr/local/openresty/nginx/lua
```

> This ensures NGINX can read and execute Lua scripts correctly.

***

### 12. Docker Compose Integration

```yaml
services:
  api-gateway:
    build:
      context: ./api-gateway
    ports:
      - "80:80"
    depends_on:
      - portal-api
      - ml-api
      - iam-api-gateway
```

***

### 13. Summary Table

| Category        | Detail                                                |
| --------------- | ----------------------------------------------------- |
| Gateway Type    | Reverse Proxy (NGINX + Lua)                           |
| Auth Method     | Lua + IAM internal validation                         |
| CORS            | Handled in Lua scripts                                |
| Error Handling  | Custom JSON errors                                    |
| Services Routed | portal-api, ml-api, iam-api-gateway                   |
| Containerized   | via Docker & Docker Compose                           |
| Extensible      | Easy to add services or Lua files                     |
| Best For        | Token auth, microservice routing, API standardization |

***

### 14. Tips & Best Practices

* Use `internal` directive for subroutes to secure them
* Test NGINX config changes with `openresty -t`
* Restart container or run `nginx -s reload` after changes
* Keep configs modular (`api.conf`, `servers.conf`, `json_errors.conf`)
* Use Lua only for logic, not heavy processing

