# json.nelua

A JSON library for the [nelua](https://nelua.io) programming language.

## Table of Contents
- [Features](#features)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Reference](#reference)
- [Error Handling](#error-handling)
- [Development](#development)

## Features
- Parse JSON strings and files into native nelua data structures
- Serialize compatible records into JSON strings
- Parsing directly into nelua records
- Support for all standard JSON types:
  - Objects (key-value pairs)
  - Arrays
  - Strings
  - Numbers
  - Booleans
  - Null

## Quick Start

```lua
local json = require "path.to.json"

-- Parse JSON directly into a record
local User = @record{
  name: string,
  age: integer,
  scores: sequence(number)
}

local json_str = [[
{
  "name": "John",
  "age": 30,
  "scores": [95.5, 88.0, 92.5]
}
]]

local user, err = json.parse_string_to_record(json_str, User)
if err ~= "" then
  print("Failed to parse:", err)
  return
end

print(user.name)    --> prints "John"
print(user.age)     --> prints 30
print(user.scores[1]) --> prints 95.5
```

## Installation

### Setup
1. Copy the `json.nelua` file into your project
2. Require the library:
```lua
local json = require "path.to.json"
```
## Reference

### JsonNodeType enum

```lua
local JsonNodeType = @enum{
  NULL = 0,
  BOOLEAN,
  STRING,
  NUMBER,
  ARRAY,
  OBJECT
}
```

### JsonNode record

Representing an individual json node after parsing

```lua
local JsonNode = @record{
  val: union{
    bol: boolean, -- accounts for NULL
    str: string,
    num: number,
    arr: sequence(JsonNode),
    obj: hashmap(string, JsonNode)
  },
  type: JsonNodeType
}
```

Example usage:

```lua
if node:is_obj() then
  local obj = node:get_obj()
  -- Work with object
elseif node:is_arr() then
  local arr = node:get_arr()
  -- Work with array
elseif node:is_str() then
  local str = node:get_str()
  -- Work with string
elseif node:is_num() then
  local num = node:get_num()
  -- Work with number
elseif node:is_bool() then
  local bool = node:get_bool()
  -- Work with boolean
elseif node:is_null() then
  -- Handle null case
end
```

### JsonNode:is

Returns a string stating the node type

```lua
function JsonNode:is(): string
```

### JsonNode:is_obj

Returns true if node is an object

```lua
function JsonNode:is_obj(): boolean
```

### JsonNode:is_num

Returns true if node is a number

```lua
function JsonNode:is_num(): boolean
```

### JsonNode:is_str

Returns true if node is a string

```lua
function JsonNode:is_str(): boolean
```

### JsonNode:is_arr

Returns true if node is an array

```lua
function JsonNode:is_arr(): boolean
```

### JsonNode:is_bool

Returns true if node is a boolean

```lua
function JsonNode:is_bool(): boolean
```

### JsonNode:is_null

Returns true if node is a null value

```lua
function JsonNode:is_null(): boolean
```

### JsonNode:get_obj

Returns a hashmap of strings to JsonNodes from the JsonNode

```lua
function JsonNode:get_obj(): hashmap(string, JsonNode)
```

### JsonNode:get_num

Returns a number from the JsonNode

```lua
function JsonNode:get_num(): number
```

### JsonNode:get_str

Returns a string from the JsonNode

```lua
function JsonNode:get_str(): string
```

### JsonNode:get_arr

Returns an array from the JsonNode

```lua
function JsonNode:get_arr(): sequence(JsonNode)
```

### JsonNode:get_bool

Returns a boolean from the JsonNode

```lua
function JsonNode:get_bool(): boolean
```

### JsonNode:get_null

Returns a null value(false) from the JsonNode

```lua
function JsonNode:get_null(): boolean
```

### json record

```lua
local json = @record{}
```

### json.parse_file

Parses a JSON file into a JsonNode structure.

```lua
local json = require "path.to.json"

local node, err = json.parse_file("config.json")
if err ~= "" then
  print("Parse error:", err)
  return
end

-- Access the node data
if node:is_obj() then
  local obj = node:get_obj()
  -- Process object
end
```

```lua
function json.parse_file(file_path: string): (JsonNode, string)
```

### json.parse_string

Parses a JSON string into a JsonNode structure.

```lua
local json = require "path.to.json"

local content = [=[
{
  "name": "John",
  "active": true,
reference "data": [1, 2, 3]
}
]=]

local node, err = json.parse_string(content)
if err ~= "" then
  print("Parse error:", err)
  return
end
```

```lua
function json.parse_string(content: string): (JsonNode, string)
```

### json.parse_string_to_record

Directly parses a JSON string into a nelua record.

```lua
local json = require "path.to.json"

local Config = @record{
  debug: boolean,
  port: integer,
  hosts: sequence(string),
  metadata: record{
    version: string,
    updated_at: string
  }
}

local content = [=[
{
  "debug": true,
  "port": 8080,
  "hosts": ["localhost", "127.0.0.1"],
  "metadata": {
    "version": "1.0.0",
    "updated_at": "2024-02-12"
  }
}
]=]

local config, err = json.parse_string_to_record(content, Config)
if err ~= "" then
  print("Failed to parse config:", err)
  return
end
```

Type Mapping:
- JSON object -> record
- JSON array -> any contiguous data structure
- JSON string -> string
- JSON number -> number
- JSON boolean -> boolean
- JSON null -> false

```lua
function json.parse_string_to_record(content: string, rec: type): (#[rec.value]#, string)
```

### json.parse_file_to_record

Directly parses a JSON file into a nelua record, see [json.parse_string_to_record](#jsonparse_string_to_record) for more information

```lua
function json.parse_file_to_record(file_path: string, rec: type)
```

### json.serialize_record

Converts a nelua record into a JSON string.

```lua
local json = require "path.to.json"

local User = @record{
  name: string,
  age: integer,
  active: boolean,
  tags: sequence(string)
}

local user: User = {
  name = "Alice",
  age = 25,
  active = true,
  tags = {"admin", "staff"}
}

local json_str = json.serialize_record(user)
print(json_str)
-- Prints {"name": "Alice", "age": 25, "active": true, "tags": ["admin", "staff"]}
```

NB: All record fields must be of supported types

```lua
function json.serialize_record(rec: auto): string
```

### json.pretty_serialize_record

Converts a nelua record into a pretty JSON string.

```lua
local json = require "path.to.json"

local User = @record{
  name: string,
  age: integer,
  active: boolean,
  tags: sequence(string)
}

local user: User = {
  name = "Alice",
  age = 25,
  active = true,
  tags = {"admin", "staff"}
}

local json_str = json.pretty_serialize_record(user, 1)
print(json_str)
--[=[Prints
{
    "name": "Alice",
    "age": 25,
    "active": true,
    "tags": [
        "admin",
        "staff"
    ]
}
]=]
```

NB: All record fields must be of supported types

```lua
function json.pretty_serialize_record(rec: auto, indent: uinteger): string
```

## Error Handling

The library handles errors by returning a string describing the error
If the string is not empty, an error has occured

```lua
local json = require "path.to.json"

-- Always check the err value
local result, err = json.parse_string(content)
if err ~= "" then
  -- Handle error case
  print("Error:", err)
  return
end

-- Type validation
if not result:is_obj() then
  print("Expected object, got:", result:is())
  return
end

-- Safe value access
local obj = result:get_obj()
```

## Development

### Running Tests
From the project root:
```console
$ nelua test.nelua
```

### Contributing
1. Fork the repository
2. Create a feature branch
3. Add tests for new features
4. Submit a pull request

For questions or issues, please use the GitHub issue tracker.
