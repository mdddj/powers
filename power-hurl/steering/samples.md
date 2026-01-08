# Hurl Samples & Patterns

## Getting Data

### Simple GET

```hurl
GET https://example.org/api/users
HTTP 200
```

### With Headers

```hurl
GET https://example.org/api/users
User-Agent: MyApp/1.0
Accept: application/json
Accept-Language: en-US
HTTP 200
```

### With Query Parameters

```hurl
GET https://example.org/api/users
[Query]
page: 1
limit: 10
sort: name
order: asc
HTTP 200
```

### Basic Authentication

```hurl
GET https://example.org/api/protected
[BasicAuth]
admin: secret123
HTTP 200
```

### Bearer Token

```hurl
GET https://example.org/api/protected
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
HTTP 200
```

### Passing Data Between Requests

```hurl
# Login and get token
POST https://example.org/api/login
{
    "username": "alice",
    "password": "secret"
}
HTTP 200
[Captures]
token: jsonpath "$.access_token"

# Use token in next request
GET https://example.org/api/profile
Authorization: Bearer {{token}}
HTTP 200
```

## Sending Data

### HTML Form Data

```hurl
POST https://example.org/login
[Form]
username: alice
password: secret123
remember_me: true
HTTP 302
```

### Multipart Form Data

```hurl
POST https://example.org/upload
[Multipart]
file: file,document.pdf; application/pdf
title: My Document
description: Important file
HTTP 201
```

### JSON Body

```hurl
POST https://example.org/api/users
Content-Type: application/json
{
    "name": "Alice",
    "email": "alice@example.org",
    "roles": ["user", "admin"]
}
HTTP 201
```

### Templated JSON Body

```hurl
POST https://example.org/api/users
Content-Type: application/json
{
    "name": "{{user_name}}",
    "email": "{{user_email}}",
    "created_at": "{{now}}"
}
HTTP 201
```

### XML Body

```hurl
POST https://example.org/api/data
Content-Type: application/xml
<?xml version="1.0" encoding="UTF-8"?>
<user>
    <name>Alice</name>
    <email>alice@example.org</email>
</user>
HTTP 201
```

### GraphQL Query

~~~hurl
POST https://example.org/graphql
Content-Type: application/json
```graphql
{
    user(id: "1") {
        name
        email
        posts {
            title
        }
    }
}
```
HTTP 200
[Asserts]
jsonpath "$.data.user.name" == "Alice"
~~~

### GraphQL with Variables

```hurl
POST https://example.org/graphql
Content-Type: application/json
{
    "query": "query GetUser($id: ID!) { user(id: $id) { name email } }",
    "variables": { "id": "1" }
}
HTTP 200
```

## Testing Responses

### Status Code

```hurl
GET https://example.org/api/users/999
HTTP 404

GET https://example.org/api/users
HTTP *
[Asserts]
status >= 200
status < 300
```

### Response Headers

```hurl
GET https://example.org/api/users
HTTP 200
Content-Type: application/json
[Asserts]
header "X-Request-Id" exists
header "Cache-Control" contains "max-age"
```

### REST API Testing

```hurl
GET https://example.org/api/orders/123
HTTP 200
[Asserts]
jsonpath "$.id" == 123
jsonpath "$.status" == "completed"
jsonpath "$.items" count >= 1
jsonpath "$.total" > 0
jsonpath "$.created_at" isIsoDate
jsonpath "$.customer.email" matches /@/
```

### HTML Response Testing

```hurl
GET https://example.org/
HTTP 200
[Asserts]
xpath "//title" exists
xpath "normalize-space(//h1)" == "Welcome"
xpath "count(//nav//a)" >= 5
xpath "//form[@id='login']" exists
```

### Cookie Testing

```hurl
GET https://example.org/login
HTTP 200
[Asserts]
cookie "session" exists
cookie "session[HttpOnly]" exists
cookie "session[Secure]" exists
cookie "session[SameSite]" == "Strict"
```

### Binary Content

```hurl
GET https://example.org/image.png
HTTP 200
[Asserts]
header "Content-Type" == "image/png"
bytes startsWith hex,89504e47;  # PNG magic bytes
```

### SSL Certificate

```hurl
GET https://example.org
HTTP 200
[Asserts]
certificate "Subject" contains "example.org"
certificate "Expire-Date" daysAfterNow > 30
```

### Full Body Check

```hurl
GET https://example.org/api/config
HTTP 200
file,expected-config.json;
```

## Common Patterns

### CSRF Token Flow

```hurl
# Get page with CSRF token
GET https://example.org/form
HTTP 200
[Captures]
csrf_token: xpath "string(//input[@name='_csrf']/@value)"

# Submit form with token
POST https://example.org/submit
[Form]
_csrf: {{csrf_token}}
name: Alice
email: alice@example.org
HTTP 302
```

### Pagination Testing

```hurl
# First page
GET https://example.org/api/items
[Query]
page: 1
limit: 10
HTTP 200
[Captures]
total: jsonpath "$.total"
[Asserts]
jsonpath "$.items" count == 10
jsonpath "$.page" == 1

# Second page
GET https://example.org/api/items
[Query]
page: 2
limit: 10
HTTP 200
[Asserts]
jsonpath "$.items" count <= 10
jsonpath "$.page" == 2
```

### Redirect Testing

```hurl
# Test redirect without following
GET https://example.org/old-page
HTTP 301
[Asserts]
header "Location" == "https://example.org/new-page"

# Test redirect with following
GET https://example.org/old-page
[Options]
location: true
HTTP 200
[Asserts]
url == "https://example.org/new-page"
```

### Polling / Retry

```hurl
# Create async job
POST https://example.org/api/jobs
{
    "type": "export"
}
HTTP 202
[Captures]
job_id: jsonpath "$.id"

# Poll until complete
GET https://example.org/api/jobs/{{job_id}}
[Options]
retry: 10
retry-interval: 1000ms
HTTP 200
[Asserts]
jsonpath "$.status" == "completed"
```

### Performance Testing

```hurl
GET https://example.org/api/fast-endpoint
HTTP 200
[Asserts]
duration < 100

GET https://example.org/api/slow-endpoint
HTTP 200
[Asserts]
duration < 5000
```

### Error Response Testing

```hurl
# Test 400 Bad Request
POST https://example.org/api/users
Content-Type: application/json
{
    "email": "invalid-email"
}
HTTP 400
[Asserts]
jsonpath "$.error" exists
jsonpath "$.message" contains "email"

# Test 401 Unauthorized
GET https://example.org/api/protected
HTTP 401

# Test 404 Not Found
GET https://example.org/api/users/99999
HTTP 404
[Asserts]
jsonpath "$.error" == "not_found"

# Test 500 Internal Server Error
POST https://example.org/api/crash
HTTP 500
```

## Reports

### HTML Report

```shell
hurl --test --report-html build/report/ *.hurl
```

### JSON Report

```shell
hurl --test --report-json build/report/ *.hurl
```

### JUnit Report (CI/CD)

```shell
hurl --test --report-junit build/report.xml *.hurl
```

### TAP Report

```shell
hurl --test --report-tap build/report.txt *.hurl
```
