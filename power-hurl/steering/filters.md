# Hurl Filters Reference

Filters transform captured or asserted values. They can be chained.

## Syntax

```hurl
jsonpath "$.value" filter1 filter2 == expected
```

## String Filters

### count

Returns the number of items in a collection or characters in a string:

```hurl
[Asserts]
jsonpath "$.items" count == 10
jsonpath "$.name" count == 5
header "Vary" count == 2
```

### nth

Returns the nth element (0-indexed):

```hurl
[Asserts]
jsonpath "$.items" nth 0 == "first"
jsonpath "$.items" nth 2 == "third"
```

### regex

Extracts value using regex with capture group:

```hurl
[Asserts]
body regex "(\\d+)" == "123"
header "Content-Type" regex "charset=([\\w-]+)" == "utf-8"
```

### replace

Replaces occurrences of a pattern:

```hurl
[Asserts]
jsonpath "$.text" replace "foo" "bar" == "bar baz"
jsonpath "$.path" replace "/" "-" == "a-b-c"
```

### split

Splits string by delimiter:

```hurl
[Asserts]
header "Content-Type" split ";" nth 0 == "application/json"
jsonpath "$.tags" split "," count == 3
```

### toString

Converts value to string:

```hurl
[Asserts]
jsonpath "$.id" toString == "123"
jsonpath "$.active" toString == "true"
```

### toInt

Converts value to integer:

```hurl
[Asserts]
jsonpath "$.count" toInt == 42
header "Content-Length" toInt > 0
```

### toFloat

Converts value to float:

```hurl
[Asserts]
jsonpath "$.price" toFloat > 9.99
version toFloat >= 1.1
```

## Encoding Filters

### base64Encode

Encodes to Base64:

```hurl
[Asserts]
body base64Encode == "SGVsbG8gV29ybGQ="
```

### base64Decode

Decodes from Base64:

```hurl
[Asserts]
jsonpath "$.encoded" base64Decode == "Hello World"
```

### urlEncode

URL encodes a string:

```hurl
[Asserts]
jsonpath "$.query" urlEncode == "hello%20world"
```

### urlDecode

URL decodes a string:

```hurl
[Asserts]
jsonpath "$.encoded" urlDecode == "hello world"
```

### htmlEscape

Escapes HTML special characters:

```hurl
[Asserts]
jsonpath "$.text" htmlEscape == "&lt;script&gt;"
```

### htmlUnescape

Unescapes HTML entities:

```hurl
[Asserts]
jsonpath "$.text" htmlUnescape == "<script>"
```

### decode

Decodes bytes with specified encoding:

```hurl
[Asserts]
bytes decode "utf-8" contains "Hello"
bytes decode "gb2312" contains "你好"
```

## Date Filters

### toDate

Parses string to date:

```hurl
[Asserts]
jsonpath "$.created" toDate "%Y-%m-%d" exists
header "Last-Modified" toDate "%a, %d %b %Y %H:%M:%S GMT" exists
```

### format

Formats date to string:

```hurl
[Captures]
date: jsonpath "$.timestamp" toDate "%Y-%m-%dT%H:%M:%SZ"
[Asserts]
variable "date" format "%Y-%m-%d" == "2024-01-15"
```

### daysAfterNow

Returns days between date and now (positive if date is in future):

```hurl
[Asserts]
certificate "Expire-Date" daysAfterNow > 30
jsonpath "$.expires" toDate "%Y-%m-%d" daysAfterNow > 7
```

### daysBeforeNow

Returns days between date and now (positive if date is in past):

```hurl
[Asserts]
jsonpath "$.created" toDate "%Y-%m-%d" daysBeforeNow < 30
certificate "Start-Date" daysBeforeNow < 365
```

## Query Filters

### jsonpath

Applies JSONPath query to value:

```hurl
[Asserts]
body jsonpath "$.users[0].name" == "Alice"
```

### xpath

Applies XPath query to value:

```hurl
[Asserts]
body xpath "//title" == "Hello"
```

## Chaining Filters

Filters can be chained left to right:

```hurl
[Asserts]
# Split, get first, trim
header "Content-Type" split ";" nth 0 == "application/json"

# Decode, then check content
bytes decode "utf-8" contains "Hello"

# Parse date, check days
jsonpath "$.expires" toDate "%Y-%m-%d" daysAfterNow > 7

# Multiple transformations
jsonpath "$.data" base64Decode toString contains "secret"
```

## Filter Examples

### Parsing Content-Type

```hurl
GET https://example.org/api
HTTP 200
[Asserts]
header "Content-Type" split ";" nth 0 == "application/json"
header "Content-Type" split ";" nth 1 replace "charset=" "" == "utf-8"
```

### Date Validation

```hurl
GET https://example.org/api/user
HTTP 200
[Asserts]
jsonpath "$.created_at" toDate "%Y-%m-%dT%H:%M:%SZ" daysBeforeNow < 365
jsonpath "$.expires_at" toDate "%Y-%m-%dT%H:%M:%SZ" daysAfterNow > 0
```

### Certificate Expiry

```hurl
GET https://example.org
HTTP 200
[Asserts]
certificate "Expire-Date" daysAfterNow > 30
certificate "Start-Date" daysBeforeNow < 730
```

### Encoded Content

```hurl
GET https://example.org/api/encoded
HTTP 200
[Asserts]
jsonpath "$.base64_data" base64Decode contains "secret"
jsonpath "$.url_encoded" urlDecode == "hello world"
```

### Collection Operations

```hurl
GET https://example.org/api/items
HTTP 200
[Asserts]
jsonpath "$.items" count == 10
jsonpath "$.items" nth 0 jsonpath "$.id" == 1
jsonpath "$.items" nth -1 jsonpath "$.id" == 10
```
