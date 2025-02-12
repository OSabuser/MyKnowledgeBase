---
title: "JSON cheatsheet complete tutorial with examples - w3schools"
source: "https://www.w3schools.io/file/json-cheatsheet/"
author:
  - "[[Renuka]]"
published: 2023-12-31
created: 2025-02-12
description: "This tutorial covers JSON cheatsheet with examples of empty json null json 2-dimensional arrays nested object array of objects sample json."
tags:
  - "clippings"
---
This tutorial covers JSON cheatsheet with examples of empty json null json 2-dimensional arrays nested object array of objects sample json..

JSON Object starts with `{` and ends with `}`. It contains an array that enclosed array of values separated by a comma in `[]`

The JSON object contains `key` and `value` pairs Syntax

```json
{
    key:value
}
```

`Key` follows below

- It must be `string` or `ASCII strings`
- Key’s first character is a letter or underscore(\_) and a dollar symbol($)
- keys are enclosed in double quotes.

`value` can be of one of the below types

- object: contains key and value pair enclosed in `{}`
- array: Contains an array of values enclosed in `[]`
- number: Contains integers, real, scientific numbers. not support Hexa and octal numbers
- string: Contains characters enclosed in double or single quote
- Boolean values are false, true only
- null

Let’s see some examples.

## How to create a null json Object

Following are examples

- Empty JSON object
- JSON object with key and null values
- JSON object with an empty array

Empty jSON objects contain null or no data and must enclose `{}` without data.

Let’s see an empty json object.

```json
{

}
```

Following is an example for JSOn with key and null value

```json
{
    "roles": null
}
```

If you want to have an empty collection of values

```json
{
    "permissions": []
}
```

## JSON with Array of objects example

The object contains key and value pairs and the Array contains a group of objects enclosed in a square bracket\[\].

```json
{
  "roles": [
    {"name": "admin", "id": 1\`},
    {"name": "sales", "id": 2}
  ]
}
```

## 2 dimensional Array JSON example

Two-dimensional arrays are an array of arrays. It can be represented as a matrix with rows and columns.

Following is a Two-dimensional array JSON example

```json
{
  "grid": [
    [1,2],
    [3,4],
    [5,6],
    [7,8,9]
  ]
}
```

## JSON nested Object example

Nested objects are parent contains child objects.

The object contains a key and the value is again one more object.

Here is an example of a JSOn nested object

```json
{
  "JOHN": {
    "name": "john",
    "active": false,
    "age": 30,
    "roles": [
      {
        "name": "admin",
        "id": 1
      },
      {
        "name": "sales",
        "id": 2
      }
    ]
  }
}
```

## Sample JSON Example

Here is a complete JSOn example

```json
{
  "name": "john",
  "active": false,
  "age": 30,
  "roles": [
    {
      "name": "admin",
      "id": 1
    },
    {
      "name": "sales",
      "id": 2
    }
  ],
  "permissions": [
    "READ",
    "WRITE",
    "COPY"
  ],
  "children's": {

  }
}
```