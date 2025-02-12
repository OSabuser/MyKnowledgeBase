---
title: "Learn JSON sample examples - w3schools"
source: "https://www.w3schools.io/json-sample-examples/"
author:
  - "[[Renuka]]"
published: 2023-12-31
created: 2025-02-12
description: "This tutorial covers showing multiple examples of demonstration of datatypes and arrays and objects in JSON data with examples."
tags:
  - "clippings"
---
This tutorial covers showing multiple examples of demonstration of datatypes and arrays and objects in JSON data with examples..

This tutorial showcases possible valid json format examples and explains different datatypes supported by key and value pairs

- JSON With String numbers Key always strings Values can be strings or numbers or any supported datatype. Strings contain readable text, enclosed in double quotes. numbers are integers or floating values.

name is a string value, salary is the number

```json
{
  "name": "Eric",
  "salary": 5000
  "department": null
}
```

`null` represents an absence of a value.

- Basic Simple Object

The object contains key and value pairs enclosed in double quotes for strings.

It is an unordered key and value pair

```json
{
  "name": "John",
  "salary": 1,
  "department": "sales",
  "active": true
}
```

- JSON Example contains an array of elements The array contains a group of elements under a single key. elements are separated by commas enclosed in Square brackets`[]`. The elements are ordered.

```json
{
  "roles": ["admin", "employee", "user", "visitor"]
}
```

- Nested JSON Object The object contains a key and value. value can be nested objects or arrays.

In this example, the Object contains a nested array and objects.

```json
{
   "name":"John",
   "salary":1,
   "department":"sales",
   "active": true,
   "roles":[
      "admin",
      "employee",
      "user",
      "visitor"
   ],
   "permanentaddress":{
      "city":"Texas",
      "zipcode":"12345"
   },
   "temporary address":{
      "city":"Denver",
      "zipcode":"84321"
   }
}
```

- JSON with an array of objects

An array of elements is enclosed in `[]`, where each element is a scalar value or an object

```json
{
   "employees":[
      {
         "name":"John",
         "salary":1,
         "department":"sales",
         "active": true
      },{
         "name": "Eric",
         "salary":12,
         "department": "marketing",
         "active": true
      },
      {
         "name": "David",
         "salary":2,
         "department": "HR",
         "active": true
      }
   ]
}
```

- Nested Arrays The array contains nested arrays. It represents rows and column values.

```json
{
    "nestedarray": [
        [
            1,
            2
        ],
        [
            4,
            5
        ],
        [
            7,
            8
        ]
    ]
}
```

- JSON object with Boolean values

```json
{
  "ssl_enabled": true,
  "status": false,
  "disabled": true
}
```

- JSON Object with Date and time createdAt and updateAt are date values enclosed in Double quotes.

Date and time data are in ISO 8601 format.

```json
{
    "name": "John",
    "salary": 1,
    "department": "sales",
    "active": true,
    "createdAt": "2022-01-11T07:22:19Z",
    "updatedAt": "2023-04-03T08:11:23Z"
}
```

- JSON with Empty Object and Array, values

The array contains empty values using square brackets `[]`. Object contains empty fields using `{}`

null value indicates no present value for a given key.

```json
{
  "array": [],
  "object": {},
  "absensevalue": null
}
```

- JSON with escape Characters

Values contain special characters that we need to escape.

```json
{
  "key": "Special escape character \"example\" , tab \t new line \n"
}
```

Double quotes are escaped using `\`. Also, you can add tabs and new line characters.

Notes:

- No whitespaces in keys allowed
- Keys are always enclosed in double quotes
- key and values are not single quotes
- an object contains a root element enclosed in `{}`