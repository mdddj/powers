# Hurl Request Reference

## Request Structure

A request consists of:
1. Method and URL (mandatory)
2. Headers (optional)
3. Sections: Options, Query, Form, Multipart, Cookies, BasicAuth (optional)
4. Body (optional)

```hurl
POST https://example.org/api/users
User-Agent: My App
Content-Type: application/json
[Options]
retry: 3
[Query]
version: 2
{
    "name": "Alice"
}
```

## HTTP Methods

All standard HTTP methods are supported:
- GET, POST, PUT, PATCH, DELETE
- HEAD, OPTIONS, CONNECT, TRACE

```hurl
GET https://example.org/resource
POST https://example.org/resource
PUT https://example.org/resource/1
PATCH https://example.org/resource/1
DELETE https://example.org/resource/1
```

## Headers

Headers follow directly after method and URL:

```hurl
GET https://example.org/api
User-Agent: Mozilla/5.0
Accept: application/json
Accept-Language: en-US
Authorization: Bearer token123
```

## Query Parameters

Use `[Query]` section for cleaner query strings:

```hurl
# URL with query string
GET https://example.org/search?q=hurl&page=1

# Equivalent using [Query] section
GET https://example.org/search
[Query]
q: hurl
page: 1
```

## Form Parameters

For `application/x-www-form-urlencoded`:

```hurl
POST https://example.org/login
[Form]
username: alice
password: secret123
remember: true
```

## Multipart Form Data

For file uploads:

```hurl
POST https://example.org/upload
[Multipart]
file: file,document.pdf;
name: my-document
description: Important file

# With explicit content type
POST https://example.org/upload
[Multipart]
file: file,image.png; image/png
```

## Cookies

```hurl
GET https://example.org/dashboard
[Cookies]
session: abc123
preferences: dark-mode
```

## Basic Authentication

```hurl
GET https://example.org/protected
[BasicAuth]
username: password

# Equivalent header
GET https://example.org/protected
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

## Request Body

### JSON Body

```hurl
POST https://example.org/api/users
{
    "name": "Alice",
    "email": "alice@example.org",
    "roles": ["admin", "user"]
}
```

### XML Body

```hurl
POST https://example.org/api/data
<?xml version="1.0"?>
<user>
    <name>Alice</name>
    <email>alice@example.org</email>
</user>
```

### GraphQL Query

~~~hurl
POST https://example.org/graphql
```graphql
{
    user(id: "1") {
        name
        email
    }
}
```
~~~

### Multiline String Body

~~~hurl
POST https://example.org/api/text
Content-Type: text/plain
```
This is a multiline
text body that preserves
line breaks.
```
~~~

### File Body

```hurl
POST https://example.org/upload
Content-Type: application/octet-stream
file,data.bin;
```

### Base64 Body

```hurl
POST https://example.org/binary
base64,SGVsbG8gV29ybGQh;
```

### Hex Body

```hurl
POST https://example.org/binary
hex,48656c6c6f;
```

## Request Options

```hurl
GET https://example.org
[Options]
# Follow redirects
location: true
location-trusted: true
max-redirs: 10

# SSL/TLS
insecure: true
cacert: /path/to/ca.pem
cert: /path/to/client.pem
key: /path/to/client.key

# Timing
delay: 1s
connect-timeout: 30s

# Retry
retry: 5
retry-interval: 500ms

# Protocol
http3: true
ipv6: true

# Output
output: response.json
verbose: true
very-verbose: true

# Authentication
user: alice:secret

# Proxy
proxy: http://proxy:8080

# Variables
variable: env=production
```

## Templates in Requests

Use `{{variable}}` for dynamic values:

```hurl
GET https://{{host}}/api/users/{{user_id}}
Authorization: Bearer {{token}}
HTTP 200
```
