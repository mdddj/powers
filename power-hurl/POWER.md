---
name: "hurl"
displayName: "Hurl HTTP Testing"
description: "Run and test HTTP requests with plain text. Use this power when working with HTTP testing, API testing, or writing .hurl files."
keywords: ["hurl", "http", "api", "test", "testing", "request", "response", "rest", "soap", "graphql", "curl", "接口测试", "HTTP测试", "API测试"]
---

# Hurl HTTP Testing Power

Hurl is a command line tool that runs HTTP requests defined in a simple plain text format. It can chain requests, capture values and evaluate queries on headers and body response.

## When to Use This Power

This power should be activated when:
- Working with `.hurl` files
- Testing HTTP endpoints and APIs
- Writing API integration tests
- Debugging HTTP requests/responses
- Learning Hurl best practices

## Quick Start

### Installation

```shell
# macOS
brew install hurl

# Linux (Debian/Ubuntu)
curl --location --remote-name https://github.com/Orange-OpenSource/hurl/releases/download/6.1.0/hurl_6.1.0_amd64.deb
sudo apt install ./hurl_6.1.0_amd64.deb

# Windows
choco install hurl
# or
winget install hurl

# Cargo
cargo install hurl
```

### Basic Usage

```shell
# Run a single file
hurl test.hurl

# Run tests with report
hurl --test --report-html build/report/ *.hurl

# Run with variables
hurl --variable host=example.net test.hurl
```

## Hurl File Format

A Hurl file consists of one or more entries, each entry being an HTTP request optionally followed by response assertions.

### Simple GET Request

```hurl
GET https://example.org/api/users
HTTP 200
```

### Request with Headers and Assertions

```hurl
GET https://example.org/api/users/1
User-Agent: My User Agent
Accept: application/json
HTTP 200
[Asserts]
jsonpath "$.name" == "John"
jsonpath "$.email" contains "@"
```

### POST with JSON Body

```hurl
POST https://example.org/api/users
Content-Type: application/json
{
    "name": "John Doe",
    "email": "john@example.org"
}
HTTP 201
[Asserts]
jsonpath "$.id" exists
```

### Capturing Values

```hurl
# Get CSRF token
GET https://example.org
HTTP 200
[Captures]
csrf_token: xpath "string(//meta[@name='_csrf_token']/@content)"

# Use captured token
POST https://example.org/login
X-CSRF-TOKEN: {{csrf_token}}
[Form]
user: alice
password: secret
HTTP 302
```

### Form Data

```hurl
POST https://example.org/contact
[Form]
name: John Doe
email: john@example.org
message: Hello World
HTTP 200
```

### Multipart Form Data

```hurl
POST https://example.org/upload
[Multipart]
file: file,data.txt;
name: my-file
HTTP 200
```

### Query Parameters

```hurl
GET https://example.org/search
[Query]
q: hurl
page: 1
limit: 10
HTTP 200
```

### Basic Authentication

```hurl
GET https://example.org/protected
[BasicAuth]
alice: secret
HTTP 200
```

### Cookies

```hurl
GET https://example.org
[Cookies]
session: abc123
theme: dark
HTTP 200
```

## Response Assertions

### Status Code

```hurl
GET https://example.org
HTTP 200

# Or with wildcard
GET https://example.org
HTTP *
[Asserts]
status >= 200
status < 300
```

### Headers

```hurl
GET https://example.org
HTTP 200
Content-Type: application/json
[Asserts]
header "Cache-Control" contains "max-age"
```

### JSONPath

```hurl
GET https://example.org/api/users
HTTP 200
[Asserts]
jsonpath "$.users" count == 10
jsonpath "$.users[0].name" == "Alice"
jsonpath "$.users[*].active" contains true
jsonpath "$.total" isInteger
```

### XPath (HTML/XML)

```hurl
GET https://example.org
HTTP 200
[Asserts]
xpath "//title" exists
xpath "normalize-space(//h1)" == "Welcome"
xpath "count(//li)" == 5
```

### Regex

```hurl
GET https://example.org/api/order
HTTP 200
[Asserts]
jsonpath "$.id" matches /^order-\d{8}$/
body matches "Hello.*World"
```

### Body

```hurl
GET https://example.org/hello
HTTP 200
[Asserts]
body contains "Hello World"
body startsWith "Hello"
bytes count == 1024
sha256 == hex,abc123...;
```

### Duration

```hurl
GET https://example.org/api
HTTP 200
[Asserts]
duration < 1000  # Response time < 1 second
```

### SSL Certificate

```hurl
GET https://example.org
HTTP 200
[Asserts]
certificate "Subject" == "CN=example.org"
certificate "Expire-Date" daysAfterNow > 30
```

## Request Options

```hurl
GET https://example.org
[Options]
location: true          # Follow redirects
insecure: true          # Allow insecure SSL
retry: 5                # Retry on failure
retry-interval: 1000ms  # Wait between retries
delay: 500ms            # Delay before request
verbose: true           # Verbose output
```

## Filters

Filters transform captured or asserted values:

```hurl
GET https://example.org/api
HTTP 200
[Asserts]
jsonpath "$.items" count == 10
jsonpath "$.name" toUpper == "ALICE"
jsonpath "$.date" toDate "%Y-%m-%d" daysBeforeNow < 7
header "Content-Type" split ";" nth 0 == "application/json"
```

## Variables

### Command Line

```shell
hurl --variable host=api.example.org --variable token=abc123 test.hurl
```

### Variables File

```shell
hurl --variables-file vars.env test.hurl
```

### Environment Variables

```shell
export HURL_host=api.example.org
hurl test.hurl
```

### In Options Section

```hurl
GET https://{{host}}/api
[Options]
variable: host=api.example.org
HTTP 200
```

## Running Tests

```shell
# Run all tests
hurl --test *.hurl

# With HTML report
hurl --test --report-html report/ *.hurl

# With JUnit report (CI/CD)
hurl --test --report-junit report.xml *.hurl

# With JSON output
hurl --json test.hurl
```

## Best Practices

1. **Organize tests by feature** - Group related tests in directories
2. **Use captures for dynamic values** - CSRF tokens, IDs, etc.
3. **Test both success and error cases** - Include 4xx/5xx tests
4. **Use variables for environment-specific values** - URLs, credentials
5. **Keep assertions focused** - Test one thing per assertion
6. **Use comments** - Document complex test scenarios

## Resources

- [Official Documentation](https://hurl.dev)
- [GitHub Repository](https://github.com/Orange-OpenSource/hurl)
- [Tutorial](https://hurl.dev/docs/tutorial/your-first-hurl-file.html)
