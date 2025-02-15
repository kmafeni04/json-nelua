# json.nelua

A JSON library for the [nelua](https://nelua.io) programming language.

## Table of Contents
- [Features](#features)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [API Reference](#api-reference)
  - [Parsing Functions](#parsing-functions)
  - [Node Operations](#node-operations)
  - [Serialization](#serialization)
- [Error Handling](#error-handling)
- [Development](#development)

## Features
- Parse JSON strings and files into native nelua data structures
- Serialize compatible records into JSON strings
- Parsing directly into nelua records
- Comprehensive error handling
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

local ok, user, err = json.parse_string_to_record(json_str, User)
if not ok then
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

## API Reference

### Parsing Functions

#### json.parse_file(file_path: string): (boolean, JsonNode, string)
Parses a JSON file into a JsonNode structure.

```lua
local json = require "path.to.json"

local ok, node, err = json.parse_file("config.json")
if not ok then
  print("Parse error:", err)
  return
end

-- Access the node data
if node:is_obj() then
  local obj = node:get_obj()
  -- Process object
end
```

#### json.parse_string(content: string): (boolean, JsonNode, string)
Parses a JSON string into a JsonNode structure.

```lua
local json = require "path.to.json"

local content = [[
{
  "name": "John",
  "active": true,
  "data": [1, 2, 3]
}
]]

local ok, node, err = json.parse_string(content)
if not ok then
  print("Parse error:", err)
  return
end
```

#### json.parse_string_to_record(content: string, rec: type): (boolean, rec, string)
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

local content = [[
{
  "debug": true,
  "port": 8080,
  "hosts": ["localhost", "127.0.0.1"],
  "metadata": {
    "version": "1.0.0",
    "updated_at": "2024-02-12"
  }
}
]]

local ok, config, err = json.parse_string_to_record(content, Config)
if not ok then
  print("Failed to parse config:", err)
  return
end
```

Type Mapping:
- JSON object → record
- JSON array → anybconiguous data structure
- JSON string → string
- JSON number → number
- JSON boolean → boolean
- JSON null → false

#### json.parse_file_to_record(file_path: string, rec: type): (boolean, rec, string)
Directly parses a JSON file into a nelua record. Takes the same arguments and follows the same type mapping as `parse_string_to_record`.

### Node Operations

After parsing JSON into a JsonNode, you can use these methods to safely interact with the data:

#### Type Checking Methods
- `node:is()`: Returns a `string` describing the node type
- `node:is_obj()`: Returns `true` if node is an object
- `node:is_arr()`: Returns `true` if node is an array
- `node:is_str()`: Returns `true` if node is a string
- `node:is_num()`: Returns `true` if node is a number
- `node:is_bool()`: Returns `true` if node is a boolean
- `node:is_null()`: Returns `true` if node is null

#### Value Access Methods
Always check the type before accessing values:

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

### Serialization

#### json.serialize_record(rec: record): string
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

#### json.pretty_serialize_record(rec: record, indent: uinteger): string
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

local json_str = json.serialize_record(user)
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

## Error Handling

The library handles errors by returning a boolean first, ensuring that users do not ignore when an error can occur:

```lua
local json = require "path.to.json"

-- Always check the ok value
local ok, result, err = json.parse_string(content)
if not ok then
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
