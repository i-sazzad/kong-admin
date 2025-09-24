# Kong-Admin
# Kong Admin API Documentation

## Table of Contents
1. [Base Configuration](#base-configuration)
2. [Admin Operations](#admin-operations)
3. [Service Management](#service-management)
4. [Route Management](#route-management)
5. [Consumer Management](#consumer-management)
6. [Plugin Management](#plugin-management)
7. [Authentication Methods](#authentication-methods)
8. [Access Control](#access-control)
9. [Response Format](#response-format)
10. [Error Codes](#error-codes)

---

## Base Configuration

**Base URL:** `http://localhost:8001`  
**Content-Type:** `application/x-www-form-urlencoded` or `application/json`  
**Authentication:** None required for Admin API

---

## Admin Operations

### Get Kong Status
Get Kong server status and health information.

**Endpoint:** `GET /status`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/status
```

**Response Example:**
```json
{
    "database": {"reachable": true},
    "memory": {
        "workers_lua_vms": [{"http_allocated_gc": "0.02 MiB", "pid": 18477}],
        "lua_shared_dicts": {
            "kong": "0.04 MiB/5.00 MiB",
            "kong_db_cache": "0.80 MiB/128.00 MiB"
        }
    },
    "server": {
        "connections_accepted": 1,
        "connections_active": 1,
        "total_requests": 1
    }
}
```

---

## Service Management

### Create Service
Create a new service in Kong.

**Endpoint:** `POST /services`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | string | Yes | Service name |
| url | string | Yes | Complete service URL |
| protocol | string | No | Protocol (http/https) |
| host | string | No | Service host |
| port | integer | No | Service port |
| path | string | No | Service path |
| connect_timeout | integer | No | Connection timeout (ms) |
| read_timeout | integer | No | Read timeout (ms) |
| write_timeout | integer | No | Write timeout (ms) |
| retries | integer | No | Number of retries |
| tags | array | No | Service tags |

**Request Example:**
```bash
curl -i -X POST http://localhost:8001/services \
  --data "name=my-service" \
  --data "url=https://api.example.com" \
  --data "connect_timeout=60000" \
  --data "read_timeout=60000" \
  --data "write_timeout=60000" \
  --data "retries=5" \
  --data "tags[]=production"
```

**Response Example:**
```json
{
    "id": "91020192-062d-416f-a275-9addeeaffaf2",
    "name": "my-service",
    "protocol": "https",
    "host": "api.example.com",
    "port": 443,
    "path": null,
    "connect_timeout": 60000,
    "write_timeout": 60000,
    "read_timeout": 60000,
    "tags": ["production"],
    "created_at": 1422386534
}
```

### List Services
Get all services with optional filtering.

**Endpoint:** `GET /services`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| tags | string | Filter by tags |
| size | integer | Number of results per page |
| offset | string | Pagination offset |

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/services
curl -i -X GET "http://localhost:8001/services?tags=production"
```

### Get Service
Get details of a specific service.

**Endpoint:** `GET /services/{service_name_or_id}`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/services/my-service
```

### Update Service
Update an existing service.

**Endpoint:** 
- `PUT /services/{service_name_or_id}` (full update)
- `PATCH /services/{service_name_or_id}` (partial update)

**Request Example:**
```bash
# Full update
curl -i -X PUT http://localhost:8001/services/my-service \
  --data "name=my-service" \
  --data "url=https://newapi.example.com"

# Partial update
curl -i -X PATCH http://localhost:8001/services/my-service \
  --data "url=https://updated-api.example.com"
```

### Delete Service
Delete a service.

**Endpoint:** `DELETE /services/{service_name_or_id}`

**Request Example:**
```bash
curl -i -X DELETE http://localhost:8001/services/my-service
```

### Get Service Plugins
List all plugins for a service.

**Endpoint:** `GET /services/{service_name_or_id}/plugins`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/services/my-service/plugins
```

---

## Route Management

### Create Route
Create a new route for a service.

**Endpoint:** `POST /services/{service_name_or_id}/routes`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | string | Yes | Route name |
| paths | array | No | Request paths |
| methods | array | No | HTTP methods |
| hosts | array | No | Host headers |
| headers | object | No | Required headers |
| strip_path | boolean | No | Strip matching path from upstream |
| preserve_host | boolean | No | Preserve original host header |
| priority | integer | No | Route priority |
| enabled | boolean | No | Route enabled status |

**Request Examples:**
```bash
# Basic route
curl -i -X POST http://localhost:8001/services/my-service/routes \
  --data "name=my-route" \
  --data "paths[]=/api"

# Route with multiple methods and paths
curl -i -X POST http://localhost:8001/services/my-service/routes \
  --data "name=api-route" \
  --data "paths[]=/api/v1" \
  --data "paths[]=/api/v2" \
  --data "methods[]=GET" \
  --data "methods[]=POST" \
  --data "methods[]=PUT"

# Host-based routing
curl -i -X POST http://localhost:8001/services/my-service/routes \
  --data "name=host-route" \
  --data "hosts[]=api.example.com" \
  --data "paths[]=/api" \
  --data "preserve_host=true"

# Header-based routing
curl -i -X POST http://localhost:8001/services/my-service/routes \
  --data "name=header-route" \
  --data "paths[]=/api" \
  --data "headers.X-API-Version=v1"
```

### List Routes
Get all routes or routes for a specific service.

**Endpoint:** 
- `GET /routes` (all routes)
- `GET /services/{service_name_or_id}/routes` (service routes)

**Request Examples:**
```bash
# All routes
curl -i -X GET http://localhost:8001/routes

# Routes for specific service
curl -i -X GET http://localhost:8001/services/my-service/routes

# Formatted route list
curl -s http://localhost:8001/routes | jq '.data[] | {name: .name, paths: .paths, methods: .methods, enabled: .enabled, service: .service.name}'
```

### Get Route
Get details of a specific route.

**Endpoint:** `GET /routes/{route_name_or_id}`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/routes/my-route
```

### Update Route
Update an existing route.

**Endpoint:**
- `PUT /routes/{route_name_or_id}` (full update)
- `PATCH /routes/{route_name_or_id}` (partial update)

**Request Examples:**
```bash
# Full update
curl -i -X PUT http://localhost:8001/routes/my-route \
  --data "name=my-route" \
  --data "paths[]=/api/v2" \
  --data "methods[]=GET"

# Partial update
curl -i -X PATCH http://localhost:8001/routes/my-route \
  --data "paths[]=/api/updated"
```

### Enable/Disable Route
Enable or disable a route.

**Endpoint:** `PATCH /routes/{route_name_or_id}`

**Request Examples:**
```bash
# Disable route
curl -i -X PATCH http://localhost:8001/routes/my-route \
  --data "enabled=false"

# Enable route
curl -i -X PATCH http://localhost:8001/routes/my-route \
  --data "enabled=true"

# Check route status
curl -i -X GET http://localhost:8001/routes/my-route | jq '.enabled'
```

### Delete Route
Delete a route.

**Endpoint:** `DELETE /routes/{route_name_or_id}`

**Request Example:**
```bash
curl -i -X DELETE http://localhost:8001/routes/my-route
```

---

## Consumer Management

### Create Consumer
Create a new consumer (API user).

**Endpoint:** `POST /consumers`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| username | string | No | Consumer username |
| custom_id | string | No | Custom identifier |
| tags | array | No | Consumer tags |

**Request Examples:**
```bash
# Basic consumer
curl -i -X POST http://localhost:8001/consumers \
  --data "username=john-doe"

# Consumer with custom ID and tags
curl -i -X POST http://localhost:8001/consumers \
  --data "username=api-user-1" \
  --data "custom_id=user-12345" \
  --data "tags[]=premium" \
  --data "tags[]=verified"
```

### List Consumers
Get all consumers with optional filtering.

**Endpoint:** `GET /consumers`

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| tags | string | Filter by tags |
| size | integer | Number of results per page |
| offset | string | Pagination offset |

**Request Examples:**
```bash
# All consumers
curl -i -X GET http://localhost:8001/consumers

# Filter by tags
curl -i -X GET "http://localhost:8001/consumers?tags=premium"

# With pagination
curl -i -X GET "http://localhost:8001/consumers?size=10"
```

### Get Consumer
Get details of a specific consumer.

**Endpoint:** `GET /consumers/{username_or_id}`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/consumers/john-doe
```

### Update Consumer
Update consumer information.

**Endpoint:**
- `PUT /consumers/{username_or_id}` (full update)
- `PATCH /consumers/{username_or_id}` (partial update)

**Request Examples:**
```bash
# Full update
curl -i -X PUT http://localhost:8001/consumers/john-doe \
  --data "username=john-doe" \
  --data "custom_id=updated-12345"

# Partial update
curl -i -X PATCH http://localhost:8001/consumers/john-doe \
  --data "tags[]=premium" \
  --data "tags[]=active"
```

### Delete Consumer
Delete a consumer.

**Endpoint:** `DELETE /consumers/{username_or_id}`

**Request Example:**
```bash
curl -i -X DELETE http://localhost:8001/consumers/john-doe
```

---

## Authentication Methods

### API Key Authentication

#### Add API Key to Consumer
**Endpoint:** `POST /consumers/{username_or_id}/key-auth`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| key | string | No | API key (auto-generated if not provided) |

**Request Examples:**
```bash
# Auto-generated key
curl -i -X POST http://localhost:8001/consumers/john-doe/key-auth

# Custom API key
curl -i -X POST http://localhost:8001/consumers/john-doe/key-auth \
  --data "key=my-secret-api-key-12345"
```

#### Enable API Key Plugin
**Endpoint:** 
- `POST /services/{service_name_or_id}/plugins` (service level)
- `POST /routes/{route_name_or_id}/plugins` (route level)

**Request Examples:**
```bash
# On service
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=key-auth" \
  --data "config.key_names[]=X-API-Key" \
  --data "config.key_names[]=apikey"

# On route
curl -i -X POST http://localhost:8001/routes/my-route/plugins \
  --data "name=key-auth" \
  --data "config.key_names[]=X-API-Key"
```

#### Test API Key
**Request Examples:**
```bash
# With header
curl -i -X GET http://localhost:8000/api/test \
  -H "X-API-Key: my-secret-api-key-12345"

# With query parameter
curl -i -X GET "http://localhost:8000/api/test?apikey=my-secret-api-key-12345"
```

### Basic Authentication

#### Add Basic Auth to Consumer
**Endpoint:** `POST /consumers/{username_or_id}/basic-auth`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| username | string | Yes | Basic auth username |
| password | string | Yes | Basic auth password |

**Request Example:**
```bash
curl -i -X POST http://localhost:8001/consumers/john-doe/basic-auth \
  --data "username=john" \
  --data "password=secret123"
```

#### Enable Basic Auth Plugin
**Request Example:**
```bash
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=basic-auth" \
  --data "config.hide_credentials=true"
```

#### Test Basic Auth
**Request Example:**
```bash
curl -i -X GET http://localhost:8000/api/test \
  -u john:secret123
```

### JWT Authentication

#### Add JWT to Consumer
**Endpoint:** `POST /consumers/{username_or_id}/jwt`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| key | string | No | JWT key identifier |
| secret | string | No | JWT secret |
| algorithm | string | No | JWT algorithm (HS256, RS256, etc.) |

**Request Examples:**
```bash
# Auto-generated
curl -i -X POST http://localhost:8001/consumers/john-doe/jwt

# Custom JWT credentials
curl -i -X POST http://localhost:8001/consumers/john-doe/jwt \
  --data "key=jwt-key-123" \
  --data "secret=my-jwt-secret" \
  --data "algorithm=HS256"
```

#### Enable JWT Plugin
**Request Example:**
```bash
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=jwt" \
  --data "config.secret_is_base64=false"
```

### OAuth 2.0 Authentication

#### Add OAuth Application to Consumer
**Endpoint:** `POST /consumers/{username_or_id}/oauth2`

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | string | Yes | Application name |
| client_id | string | No | OAuth client ID |
| client_secret | string | No | OAuth client secret |
| redirect_uris | array | Yes | Redirect URIs |

**Request Example:**
```bash
curl -i -X POST http://localhost:8001/consumers/john-doe/oauth2 \
  --data "name=My Mobile App" \
  --data "client_id=mobile-app-client" \
  --data "client_secret=mobile-app-secret" \
  --data "redirect_uris[]=https://myapp.com/callback"
```

#### Enable OAuth 2.0 Plugin
**Request Example:**
```bash
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=oauth2" \
  --data "config.scopes[]=read" \
  --data "config.scopes[]=write" \
  --data "config.mandatory_scope=true"
```

---

## Plugin Management

### Add Plugin
Add a plugin to service, route, or consumer.

**Endpoints:**
- `POST /services/{service_name_or_id}/plugins` (service level)
- `POST /routes/{route_name_or_id}/plugins` (route level)
- `POST /consumers/{username_or_id}/plugins` (consumer level)
- `POST /plugins` (global level)

### Rate Limiting Plugin

**Request Examples:**
```bash
# Service level rate limiting
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=rate-limiting" \
  --data "config.second=10" \
  --data "config.minute=100" \
  --data "config.hour=1000" \
  --data "config.day=10000" \
  --data "config.policy=local" \
  --data "config.limit_by=ip"

# Route level rate limiting
curl -i -X POST http://localhost:8001/routes/my-route/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=50" \
  --data "config.hour=500" \
  --data "config.limit_by=consumer"

# Consumer level rate limiting
curl -i -X POST http://localhost:8001/consumers/premium-user/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=1000" \
  --data "config.hour=10000"
```

### Request Termination Plugin (Enable/Disable)

**Disable Service/Route:**
```bash
# Disable service
curl -i -X POST http://localhost:8001/services/my-service/plugins \
  --data "name=request-termination" \
  --data "config.status_code=503" \
  --data "config.message=Service temporarily disabled"

# Disable route
curl -i -X POST http://localhost:8001/routes/my-route/plugins \
  --data "name=request-termination" \
  --data "config.status_code=503" \
  --data "config.message=Route temporarily disabled"
```

### List Plugins
Get plugins for specific entity.

**Request Examples:**
```bash
# Service plugins
curl -i -X GET http://localhost:8001/services/my-service/plugins

# Route plugins
curl -i -X GET http://localhost:8001/routes/my-route/plugins

# Consumer plugins
curl -i -X GET http://localhost:8001/consumers/john-doe/plugins

# All plugins
curl -i -X GET http://localhost:8001/plugins
```

### Get Plugin
Get specific plugin details.

**Endpoint:** `GET /plugins/{plugin_id}`

**Request Example:**
```bash
curl -i -X GET http://localhost:8001/plugins/{plugin-id}
```

### Update Plugin
Update plugin configuration.

**Endpoint:** `PATCH /plugins/{plugin_id}`

**Request Example:**
```bash
curl -i -X PATCH http://localhost:8001/plugins/{plugin-id} \
  --data "config.minute=200" \
  --data "config.hour=2000"
```

### Enable/Disable Plugin
Enable or disable a plugin.

**Request Examples:**
```bash
# Disable plugin
curl -i -X PATCH http://localhost:8001/plugins/{plugin-id} \
  --data "enabled=false"

# Enable plugin
curl -i -X PATCH http://localhost:8001/plugins/{plugin-id} \
  --data "enabled=true"
```

### Delete Plugin
Delete a plugin.

**Endpoint:** `DELETE /plugins/{plugin_id}`

**Request Example:**
```bash
curl -i -X DELETE http://localhost:8001/plugins/{plugin-id}
```

---

## Access Control

### ACL (Access Control Lists)

#### Assign Consumer to Group
**Endpoint:** `POST /consumers/{username_or_id}/acls`

**Request Examples:**
```bash
# Create different consumer groups
curl -i -X POST http://localhost:8001/consumers/admin-user/acls \
  --data "group=admin"

curl -i -X POST http://localhost:8001/consumers/regular-user/acls \
  --data "group=users"

curl -i -X POST http://localhost:8001/consumers/guest-user/acls \
  --data "group=guests"
```

#### Apply ACL to Routes/Services
**Request Examples:**
```bash
# Allow only admin group
curl -i -X POST http://localhost:8001/routes/admin-route/plugins \
  --data "name=acl" \
  --data "config.allow[]=admin"

# Allow multiple groups
curl -i -X POST http://localhost:8001/routes/user-route/plugins \
  --data "name=acl" \
  --data "config.allow[]=admin" \
  --data "config.allow[]=users"

# Deny specific groups
curl -i -X POST http://localhost:8001/routes/public-route/plugins \
  --data "name=acl" \
  --data "config.deny[]=banned"
```

---

## Response Format

### Success Response
```json
{
    "data": {
        "id": "91020192-062d-416f-a275-9addeeaffaf2",
        "name": "example-service",
        "created_at": 1422386534,
        "updated_at": 1422386534
    }
}
```

### List Response
```json
{
    "data": [
        {
            "id": "91020192-062d-416f-a275-9addeeaffaf2",
            "name": "example-service",
            "created_at": 1422386534
        }
    ],
    "next": "http://localhost:8001/services?offset=eyJjcmVhdGVkX2F0IjoxNDIyMzg2NTM0MDAwLCJpZCI6IjkxMDIwMTkyLTA2MmQtNDE2Zi1hMjc1LTlhZGRlZWFmZmFmMiJ9"
}
```

### Error Response
```json
{
    "message": "schema violation (name: required field missing)",
    "name": "schema violation",
    "fields": {
        "name": "required field missing"
    },
    "code": 2
}
```

---

## Error Codes

| HTTP Status | Description |
|-------------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 204 | No Content - Resource deleted successfully |
| 400 | Bad Request - Invalid request format |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Resource already exists |
| 500 | Internal Server Error - Server error |

### Common Error Messages

| Error | Description | Solution |
|-------|-------------|----------|
| `schema violation (name: required field missing)` | Required field missing | Provide the required field |
| `unique violation (name: already exists)` | Resource name already exists | Use a different name |
| `foreign key violation` | Referenced resource doesn't exist | Create the referenced resource first |
| `invalid service` | Service doesn't exist | Create the service first |

---

## Utility Scripts

### Check Plugin Status
```bash
# Get specific plugin by name
curl -s http://localhost:8001/services/my-service/plugins | jq '.data[] | select(.name=="rate-limiting")'

# Get plugin ID for updates
PLUGIN_ID=$(curl -s http://localhost:8001/services/my-service/plugins | jq -r '.data[] | select(.name=="rate-limiting") | .id')
```

### Bulk Operations
```bash
# Disable all routes for a service
SERVICE_ROUTES=$(curl -s http://localhost:8001/services/my-service/routes | jq -r '.data[].id')
for route_id in $SERVICE_ROUTES; do
    curl -i -X PATCH http://localhost:8001/routes/$route_id --data "enabled=false"
done

# Create multiple consumers
USERS=("alice" "bob" "charlie" "diana")
for user in "${USERS[@]}"; do
    curl -i -X POST http://localhost:8001/consumers \
      --data "username=$user" \
      --data "custom_id=user-$user"
    
    # Add API key for each
    curl -i -X POST http://localhost:8001/consumers/$user/key-auth
done
```

### Monitoring and Analytics
```bash
# List all consumers with their credentials
curl -s http://localhost:8001/consumers | jq -r '.data[] | .username' | while read consumer; do
    echo "Consumer: $consumer"
    echo "API Keys:"
    curl -s http://localhost:8001/consumers/$consumer/key-auth | jq -r '.data[].key // "None"'
    echo "Groups:"
    curl -s http://localhost:8001/consumers/$consumer/acls | jq -r '.data[].group // "None"'
    echo "---"
done

# Check rate limit headers in responses
curl -i http://localhost:8000/api/test | grep "X-RateLimit"
```

This documentation provides comprehensive coverage of Kong Admin API operations for managing services, routes, consumers, and plugins. All examples use standard HTTP methods and can be implemented in any programming language or framework.
