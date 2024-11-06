# protoc-gen-pydantic

`protoc-gen-pydantic` is a `protoc` plugin that automatically generates [Pydantic](https://docs.pydantic.dev/) model definitions from `.proto` files. This tool is designed to help developers seamlessly integrate protobuf-defined models with Pydantic, a powerful data validation and settings management library for Python.

## Features

- Supports all standard `proto3` field types.
- Handles nested messages and enums.
- Generates Pydantic models with type annotations and field descriptions.
- Supports `oneof`, `optional`, `repeated`, and `map` fields.
- Retains comments from `.proto` files as docstrings in the generated models.

## Installation

You can download the binaries from GitHub [Releases](https://github.com/ornew/protoc-gen-pydantic/releases).

### Build from Source

You first need to have Go installed. If you don't have Go installed, you can download it from the Go downloads page.

Clone the repository and build the plugin:

```sh
git clone https://github.com/ornew/protoc-gen-pydantic
cd protoc-gen-pydantic
go build -o protoc-gen-pydantic main.go
```

## Usage

To generate Pydantic model definitions, use `protoc` with your `.proto` files specifying `--pydantic_out`:

```sh
protoc --plugin=protoc-gen-pydantic=./protoc-gen-pydantic \
       --pydantic_out=./gen \
       --proto_path=./proto_files \
       ./api/example.proto
```

If you use [buf](https://buf.build/):

```yaml
# buf.gen.yaml
version: v2
plugins:
  - local: protoc-gen-pydantic
    opt:
      - paths=source_relative
    out: gen
inputs:
  - directory: api
```

```sh
buf config init
buf generate
```

## Example

Given a simple `.proto` file:

```proto
syntax = "proto3";

package example;

// User model representing the example.User message.
message User {
  string name = 1;
  int32 age = 2;
  repeated string emails = 3;
  bool is_active = 4;
}
```

Running `protoc` will generate a Pydantic model like this:

```python
from enum import Enum as _Enum
from typing import Optional as _Optional

from pydantic import BaseModel as _BaseModel, Field as _Field

class User(_BaseModel):
    """
    User model representing the example.User message.
    """
    name: str = _Field(...)
    age: int = _Field(...)
    emails: list[str] = _Field(...)
    is_active: bool = _Field(...)
```

## Options

| option | description |
|--------|-------------|
| `preserving_proto_field_name` | Use the proto field naming for the output field name. If `false`, it will be in caml-case according to `protojson` rules. Defaults to `false`. |
| `auto_trim_enum_prefix` | Automatically remove prefixes from enum fields. Defaults to `true` |
| `use_integers_for_enums` | Use integers for enum values instead of enum names. Defaults to `false`. |
| `disable_field_description` | Disable generating the field description. Defaults to `false`. |
| `use_none_union_syntax_instead_of_optional` | Use `T | None` instead of `Optional[T]`. |

### `auto_trim_enum_prefix`

```proto
enum Status {
  STATUS_UNSPECIFIED = 0;
  STATUS_OK = 1;
  STATUS_ERROR = 2;
}
```

If `auto_trim_enum_prefix` is `false`:

```python
class Status(str, _Enum):
    STATUS_UNSPECIFIED = "UNSPECIFIED"
    STATUS_OK = "OK"
    STATUS_ERROR = "ERROR"
```

If `auto_trim_enum_prefix` is `true` (default):

```python
class Status(str, _Enum):
    UNSPECIFIED = "UNSPECIFIED"
    OK = "OK"
    ERROR = "ERROR"
```

### `use_integers_for_enums`

If `use_integers_for_enums` is `false`:

```python
class Status(str, _Enum):
    UNSPECIFIED = "UNSPECIFIED"
    OK = "OK"
    ERROR = "ERROR"
```

If `use_integers_for_enums` is `true`:

```python
class Status(int, _Enum):
    UNSPECIFIED = 0
    OK = 1
    ERROR = 2
```

### `disable_field_description`

If `disable_field_description` is `true`:

```proto
message User {
    // User name
    string name = 1;
}
```

```python
class User(_BaseModel):
    # User name
    name: str = _Field(..., description="User name")
```

If `disable_field_description` is `false`:

```python
class User(_BaseModel):
    # User name
    name: str = _Field(...)
```

### `use_none_union_syntax_instead_of_optional`

If `use_none_union_syntax_instead_of_optional` is `true`: `Optional[T]`
If `use_none_union_syntax_instead_of_optional` is `false`: `T | None`

## Contributing

Contributions are welcome! Please open an issue or submit a pull request with your changes.

## License

This project is licensed under the Apache License 2.0. See LICENSE for more details.

## Contact

If you have any questions, feel free to reach out to the author:

- Name: Arata Furukawa
- Email: old.river.new@gmail.com
