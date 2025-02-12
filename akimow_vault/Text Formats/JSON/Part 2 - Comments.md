---
title: "How to add comments in JSON data| Multiple ways to write JSON Comments - w3schools"
source: "https://www.w3schools.io/file/json-comments/"
author:
  - "[[Renuka]]"
published: 2023-12-31
created: 2025-02-12
description: "This tutorial covers adding comments in JSON data, Does JSOn accept comments Why comments are not included in JSON data with examples?"
tags:
  - "clippings"
---
This tutorial covers adding comments in JSON data, Does JSOn accept comments Why comments are not included in JSON data with examples?.

`Comments` are a piece of text string about the line of a block of code and these are ignored by a processor of that file.

Does JSON support Comments?

As per Standard JSON rules, There is no official support for Comments in JSON content.

JSON contains keys and values as per standard rules, and these can be parsed by different parsers using programming libraries.

SO Even if json allows comments, You need to have support for the parser to read and understand the comments.

If you write like this json file with content, it is an invalid json file

```json
{
"key": "value" // comments are not allowed and invalid
}
```

You can achieve multiple ways to have comments in the JSOn object.

One way is **using adding key-value pairs with comments and descriptions** JSOn always contains data of keys and values, so add the `comments` key in json object with a value is a comment string value.

```json
{
"key": "value",
"comments":"One way to write comments here"
}
```

You need to have a parser or processor read and ignore the comments key in a code.

Second way, use [JSOn minify](https://github.com/getify/JSON.minify) library by removing the spaces and c language comments from the json document before parsing the json string

For example

```javascript
let jsonString="{
"key": "value",
 //Single-line comments
/* Multi-line comments
example
*/}"

console.log(JSON.parse(JSON.minify(jsonString)));
```

`JSON.minify` removes the white spaces and comments from json string.

Here is an output

```json
{
"key": "value",
 }
```

The third way is using JavaScript variables.

JSON is about javascript objects, so declare json data in a javascript variable and add the comments in json format as seen below.

```javascript
const jsondata = {
"name": "john",
//Comments: javascript comments in a json string
}
```

These are not pure json comments, but javascript json objects with comments that are ignored by the javascript compiler.

JSON is an open standard data exchange between Client and server which contains data of keys and values

The server produces the JSOn data and consumers or clients consume the data json from the Server, SO there is no explanation of what the is data about in the form of comments.

Another reason is if you add comments, It makes it the human and library parser to read the json data.

Comments are usually added to source code to describe lines of code. JSON is purely about the data format to send, Hence It is not feasible to add comments to JSOn

## Conclusion,

As JSON does not support comments, You can still add comments in multiple ways.