# TCSV (Typed Comma Separated Values) 1.0

## Abstract

In order to make CSV files a better portable data format, this proposal introduce the TCSV
(Typed Comma Separated Values) format, an struct pseudo-language and typing system.
It is designed to be human-readable and easier to validate than plain CSV. It is
also designed to be type-safe. TCSV can be used as a strict and a lax superset of CSV,
meaning that any CSV file must be also a valid TCSV file, using the basic type inference.

## Status of this memo

This memo is a work in progress and is not a standard. It is a
proposal draft for a new data format and is subject to change until being released.

## Draft

### Introduction

TCSV is a data format that is a superset of CSV. It is designed to easy read and
write by humans.

### File extensions

- `.tcsv`: TCSV file.
- `.tcsvh`: TCSV Header file.
- `.tcsvb`: TCSV Binary file (reserved for future use).
- `.tcsvz`: TCSV Compressed file (reserved for future use).

### Problem statement

CSV is a popular data format. It is used for many purposes, such as data
exchange, data storage, and data analysis. However, CSV has some limitations.
One of the main limitations of CSV is that it is not type-safe. This means that
it is easy to make mistakes when working with CSV files. For example, it is easy
to accidentally mix up the order of columns in a CSV file, or to accidentally
use the wrong data type for a column. These mistakes can lead to errors in data
analysis, and can be difficult to debug.

### Solution

Create a verbose, explicit-first and exportable data format that declares the type of each
column, based on static types which do not depend on implicit types, implicit metadata or exif information.

#### Key differences

1. TCSV use a header section to declare custom types.
2. TCSV use a `@` character to declare file parameters.
3. TCSV use a `:` character to separate the column name from the type.
4. Header section can be defined under a block delimited by `---` characters for
multiline support.

#### Implementation keys

1. When comments and header types are removed, the file must be a valid CSV file.
2. The header section must be defined before the data section.
3. TCSV implementations are always case-sensitive.
4. TCSV implementations are always separated by commas.
5. TCSV implementations can contain collation definition using a file parameter.
Defaults to `utf-8`.
6. Type arguments are defined inside `{` and `}` characters.
7. Type argumwnts are separated by commas.
8. Type arguments are always lowercase.

#### Reserved keywords and directives

- `@dir`: Used to define the root directory for relative paths in imports. Defaults to the current working directory.
- `@import`: Used to import external type definitions file (TCSVH). This directive must include the UNIX-like path to the file.
- `@include`: An alias of `@import`.
- `@define`: Used to define custom types.
- `@extend`: Used to extend custom types from other custom types.
- `@function`: Used to define custom functions.
- `@export`: Used to define the automatically exported columns.
- `@avoid`: Used to define the automatically avoided columns.

#### Primitive types

- `text`: A string of characters.
- `integer`: A whole number.
- `boolean`: A logical boolean value (can be also implemented with 0 and 1 values).
- `biginteger`: A whole number with arbitrary precision.
- `float`: A floating point number.

#### Built-in complex types

- `regex`: A regular expression, based on the ECMAScript standard.
- `number`: A number that can be either an integer or a float.
- `date`: A date in the ISO/IEC 8601 format.
- `time`: A time in the ISO/IEC 8601 format.
- `datetime`: A datetime in the ISO/IEC 8601 format.
- `currency`: A currency value based on the ISO/IEC 4217 standard.
- `percentage`: A float or integer number ended by `%`.

#### Built-in helper types

- `empty`: A value that is empty. It is processed as text equivalent to `""`.
- `any`: Set column as not typed. It is processed as text.
- `null`: Set column as nullable. It is processed as text equivalent to `null`.

#### Type definitions

Custom types can be defined using the `@define` directive. Custom types can be
based on primitive types or other custom types.

Types can be also imported from an external TCSVH (TCSV Header) file using the
`@import` or `@include` directives.

#### Type inference

When no header is defined, the parser must infer the type of each column using
the following rules:

1. If the value is empty, the type is `empty`, which is parsed as a zero-bytes text.
2. If the value is `true` or `false` (case-insensitive), the type is `boolean`.
3. If the value is a whole number, the type is `integer`.
4. If the value is a whole number with arbitrary precision, the type is `biginteger`.
5. If the value is a floating point number, the type is `float`.
6. If the value matches the ISO/IEC 8601 date format, the type is `date`.
7. If the value matches the ISO/IEC 8601 time format, the type is `time`.
8. If the value matches the ISO/IEC 8601 datetime format, the type is `datetime`.
9. If the value matches the ISO/IEC 4217 currency format, the type is `currency`.
10. If the value is ended by a `%` character and is a valid float or integer number, the type is `percentage`.
11. Otherwise, the type is `text`.

#### Type assignation

- When a header name is the same as a defined custom type, the custom type will be used to validate the column values.
- When a header name has no type assigned and there is no custom type defined with the same name, the type will be determined as `any`.

#### Type modification operators

- `<type>?`: Make a type optional. Which is parsed as the primitive type falsy
value or `empty` if the type is not a primitive type.
- `[<type>]`: Make a type an array. Which is parsed as a list of primitive values
separated by commas.
- `<type> | <type>`: Make a type a union of types. Which is parsed as any of the
types defined in the union.
- `<type> & <type>`: Make a type an intersection of types. Which is parsed as all
the types defined in the intersection.

#### Type rewrite

When a type is defined using the `@define` directive, it can be rewritten
using the same name in the header section.

#### Type extension

Custom types can be extended using the `@extend` directive. This directive
must be used after the `@define` directive.

```TCSV
@define int-number: integer
@extend positive-integer: int-number{min:1}
```

#### Type flags

- `length:<number>`: Define the maximum length of a text.
- `min:<number>`: Define the minimum value of a number.
- `max:<number>`: Define the maximum value of a number.
- `regex:"<pattern>"`: Define a regular expression pattern that the value must match.
- `positive`: Define that the number must be positive (greater than 0).
- `negative`: Define that the number must be negative (less than 0).
- `nonzero`: Define that the number must be non-zero (not equal to 0).
- `nonempty`: Define that the text must be non-empty (not equal to `""`).
- `case-sensitive`: Define that the text must be case-sensitive.
- `case-insensitive`: Define that the text must be case-insensitive.

#### Built-in file parameters

- `@collation`: Define the file collation. Defaults to `utf-8`.
- `@author`: Define the file author.
- `@license`: Define the file license.
- `@created`: Define the file creation date using ISO/IEC 8601 Datetime.
- `@modified`: Define the file last modification date.
- `@version`: Define the file version.
- `@comment`: Define a file comment.

#### Custom file parameters

These can be defined using the `@define` directive with the addition of an `@` symbol at the beginning of the parameter name, which **must not** contain spaces and can be defined using kebab-case.

Example:

```tcsv
@define @organization: text{length:50,regex:"^[A-Za-z0-9 ]+$"}
@define @phone-number: text{regex:"^\+?[0-9]{7,15}$"}

@organization: "TCSV Specification Sample Organization"
@phone-number: "+1234567890"
```

### Example

#### Inline header

```tcsv
@collation: "utf-8"
@author: "John Doe"
@license: "GPL-3.0"
@version: "1.0"

@define age: integer{positive}
@define pets: [text{length:25}]

name:text{length:80},age:age,pets:pets
John,25,"cat,dog"
Jane,30,"bird"
```

#### Multiline headers

In TCSV files, headers must be defined in a block delimited by `---` characters.

```tcsv
---
@collation: "utf-8"
@author: "John Doe"
@license: "GPL-3.0"
@version: "1.0"

@define age: integer{positive}
@define pets: text{length:25}[]

name: text{length:80} -- This is a comment
age: age, -- Commas are optional in multiline headers
pets: pets
does like tea: boolean?
---
John,25,"cat,dog",true
Jane,30,"bird"
```

```tcsv
---
@define @name: text{length:80}

@define id: integer{positive}

@define organization: text{
  length:50,
  regex:"^[A-Za-z0-9 ]+$"
}

@name: "TCSV Specification Sample"
@collation: "utf-8"
@author: "John Doe"
@license: "GPL-3.0"
@version: "1.0"

id: id
org: organization
phone-number: text{
  regex:"^\+?[0-9]{7,15}$"
}
---
1,"Google Inc","+1234567890"
2,"Microsoft","+19876543210"
3,"Amazon","+11234567890"
```

Header files are multiline by default, so the `---` delimiters are optional.
