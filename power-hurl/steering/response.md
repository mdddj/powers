# Hurl Response Reference

## Response Structure

A response consists of:
1. Version and Status (mandatory)
2. Headers (optional, implicit asserts)
3. Captures section (optional)
4. Asserts section (optional)
5. Body (optional, implicit assert)

```hurl
GET https://example.org/api/user
HTTP 200
Content-Type: application/json
[Captures]
user_id: jsonpath "$.id"
[Asserts]
jsonpath "$.name" exists
{
    "id": 123,
    "name": "Alice"
}
```

## Version and Status

### Explicit Version

```hurl
GET https://example.org
HTTP/1.1 200

GET https://example.org
HTTP/2 200

GET https://example.org
HTTP/3 200
```

### Any Version

```hurl
GET https://example.org
HTTP 200
```

### Wildcard Status

```hurl
GET https://example.org
HTTP *
[Asserts]
status >= 200
status < 300
```

## Implicit Header Asserts

Headers after status line are implicit asserts:

```hurl
GET https://example.org
HTTP 200
Content-Type: application/json; charset=utf-8
Cache-Control: no-cache
```

## Captures

Capture values from response for use in subsequent requests:

### Header Capture

```hurl
GET https://example.org
HTTP 200
[Captures]
content_type: header "Content-Type"
location: header "Location"
```

### Cookie Capture

```hurl
GET https://example.org/login
HTTP 200
[Captures]
session_id: cookie "SESSIONID"
session_expires: cookie "SESSIONID[Expires]"
```

### Body Capture

```hurl
GET https://example.org/text
HTTP 200
[Captures]
body_content: body
```

### JSONPath Capture

```hurl
GET https://example.org/api/user
HTTP 200
[Captures]
user_id: jsonpath "$.id"
user_name: jsonpath "$.name"
first_role: jsonpath "$.roles[0]"
all_ids: jsonpath "$.users[*].id"
```

### XPath Capture

```hurl
GET https://example.org/page
HTTP 200
[Captures]
title: xpath "string(//title)"
csrf: xpath "string(//meta[@name='csrf']/@content)"
link_count: xpath "count(//a)"
```

### Regex Capture

```hurl
GET https://example.org/page
HTTP 200
[Captures]
# Must have at least one capture group
order_id: regex "order-([0-9]+)"
date: regex "(\\d{4}-\\d{2}-\\d{2})"
```

### Status Capture

```hurl
GET https://example.org
HTTP *
[Captures]
status_code: status
```

### Duration Capture

```hurl
GET https://example.org
HTTP 200
[Captures]
response_time: duration
```

### URL Capture

```hurl
GET https://example.org/redirect
[Options]
location: true
HTTP 200
[Captures]
final_url: url
```

### Certificate Capture

```hurl
GET https://example.org
HTTP 200
[Captures]
cert_subject: certificate "Subject"
cert_expiry: certificate "Expire-Date"
```

## Asserts

### Status Assert

```hurl
GET https://example.org
HTTP *
[Asserts]
status == 200
status >= 200
status < 300
status != 404
```

### Version Assert

```hurl
GET https://example.org
HTTP *
[Asserts]
version == "2"
version startsWith "1"
```

### Header Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
header "Content-Type" == "application/json"
header "Content-Type" contains "json"
header "Cache-Control" exists
header "X-Custom" not exists
header "Vary" count == 2
```

### Cookie Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
cookie "session" exists
cookie "session[Value]" matches /^[a-f0-9]+$/
cookie "session[Expires]" exists
cookie "session[HttpOnly]" exists
cookie "session[Secure]" exists
cookie "session[SameSite]" == "Strict"
cookie "session[Path]" == "/"
cookie "session[Domain]" == "example.org"
```

### Body Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
body contains "Hello"
body startsWith "<!DOCTYPE"
body endsWith "</html>"
body matches /Hello \w+/
body == "exact content"
```

### Bytes Assert

```hurl
GET https://example.org/binary
HTTP 200
[Asserts]
bytes count == 1024
bytes startsWith hex,89504e47;  # PNG magic bytes
```

### JSONPath Assert

```hurl
GET https://example.org/api
HTTP 200
[Asserts]
# Equality
jsonpath "$.name" == "Alice"
jsonpath "$.age" == 30
jsonpath "$.active" == true
jsonpath "$.data" == null

# Existence
jsonpath "$.id" exists
jsonpath "$.deleted" not exists

# Comparison
jsonpath "$.count" > 0
jsonpath "$.count" >= 1
jsonpath "$.count" < 100
jsonpath "$.count" <= 99

# String predicates
jsonpath "$.email" contains "@"
jsonpath "$.name" startsWith "A"
jsonpath "$.name" endsWith "e"
jsonpath "$.id" matches /^[0-9a-f-]+$/

# Collection
jsonpath "$.items" count == 10
jsonpath "$.tags" contains "important"
jsonpath "$.roles" includes "admin"

# Type checks
jsonpath "$.id" isInteger
jsonpath "$.price" isFloat
jsonpath "$.active" isBoolean
jsonpath "$.name" isString
jsonpath "$.items" isCollection
jsonpath "$.created" isIsoDate
```

### XPath Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
xpath "//title" exists
xpath "//h2" not exists
xpath "normalize-space(//h1)" == "Welcome"
xpath "count(//li)" == 5
xpath "count(//li)" > 0
xpath "boolean(//form)" == true
xpath "string(//a/@href)" startsWith "https"
```

### Regex Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
# With capture group - asserts on captured value
regex "id=(\\d+)" == "123"
regex /order-(\d+)/ matches /^\d{6}$/
```

### Duration Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
duration < 1000      # Less than 1 second
duration < 500ms     # Less than 500ms
duration >= 100ms
```

### SHA-256 Assert

```hurl
GET https://example.org/file
HTTP 200
[Asserts]
sha256 == hex,e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855;
```

### MD5 Assert

```hurl
GET https://example.org/file
HTTP 200
[Asserts]
md5 == hex,d41d8cd98f00b204e9800998ecf8427e;
```

### Certificate Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
certificate "Subject" contains "example.org"
certificate "Issuer" contains "Let's Encrypt"
certificate "Expire-Date" daysAfterNow > 30
certificate "Start-Date" daysBeforeNow < 365
certificate "Serial-Number" matches /[0-9a-f]+/
```

### IP Address Assert

```hurl
GET https://example.org
HTTP 200
[Asserts]
ip startsWith "192.168"
ip isIpv4
ip isIpv6
```

### Variable Assert

```hurl
GET https://example.org/api
HTTP 200
[Captures]
items: jsonpath "$.items"
[Asserts]
variable "items" count == 10
variable "items" contains {"id": 1}
```

## Response Body

Implicit body assert (exact match):

```hurl
# JSON body
GET https://example.org/api/user
HTTP 200
{
    "id": 1,
    "name": "Alice"
}

# XML body
GET https://example.org/api/data
HTTP 200
<?xml version="1.0"?>
<user><name>Alice</name></user>

# Plain text
GET https://example.org/hello
HTTP 200
`Hello World!`

# File comparison
GET https://example.org/data
HTTP 200
file,expected.json;
```
