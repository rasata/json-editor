JSON Editor
===========

![JSON Schema -> HTML Editor -> JSON](https://raw.github.com/jdorn/json-editor/master/jsoneditor.png)

JSON Editor will create an HTML editor from a JSON Schema and output JSON data that matches the schema.
It supports a large subset of JSON Schema and can integrate with several popular CSS frameworks (bootstrap, foundation, and jQueryUI).

Check out an example: http://rawgithub.com/jdorn/json-editor/master/example.html

Download the [production version][min] or the [development version][max].

[min]: https://raw.github.com/jdorn/json-editor/master/dist/jquery.jsoneditor.min.js
[max]: https://raw.github.com/jdorn/json-editor/master/dist/jquery.jsoneditor.js

Requirements
-----------------

*  A recent version of jQuery
*  A modern browser

### Optional Requirements

*  A compatible javascript template engine (Mustache, Underscore, Hogan, Handlebars, Swig, Markup, or EJS)
*  A compatible CSS Framework for styling (bootstrap 2/3, foundation 3/4/5, or jqueryui)
*  [SCEditor](http://www.sceditor.com/) for WYSIWYG editing of HTML or BBCode
*  [EpicEditor](http://epiceditor.com/) for Markdown editing

Usage
--------------

### Initialize

```js
$("#editor_holder").jsoneditor(options);
```

#### Options

*  __schema__ - Required, a valid JSON Schema (either Version 3 or Version 4 of the draft spec).
*  __startval__ - Optional, the starting value for the editor.  Must be valid against the schema.
*  __ajax__ - Optional, if `true`, JSON Editor will load external urls in `$ref` via ajax.  Default `false`.
*  __refs__ - Optional, an object containing schema definitions for urls.  Allows you to pre-define external schemas.
*  __required_by_default__ - Optional, if `true`, all schemas that don't have the `required` property explicitly set will be required. Default `false`.
*  __theme__ - Optional, sets the CSS theme to use for the editor.  See the "CSS Integration" section below for more info.
*  __template__ - Optional, sets the js template engine to use.  See the "Templates and Variables" section below for more info.

Here's an example using all the options:

```js
$("#editor_holder").jsoneditor({
  schema: {
    type: "object",
    properties: {
      name: {
        description: "Will load from the pre-defined schema passed in during initialization",
        $ref: "http://example.com/name.json"
      },
      age: {
        description: "Will load via ajax.  If the ajax option was false, this would throw an exception",
        $ref: "http://example.com/age.json"
      }
    }
  },
  startval: {
    name: "John Smith",
    age: 21
  },
  ajax: true,
  refs: {
    "http://example.com/name.json": {
      type: "string"
    }
  },
  required_by_default: true,
  theme: 'bootstrap3',
  template: 'underscore'
});
```

__*Note__ If the `ajax` property is `true` and JSON Editor needs to fetch an external url, the api methods won't be available immediately.
Listen for the `ready` event before calling them.
```js
$("#editor_holder").on('ready',function() {
  // Now the api methods will be available
  $("#editor_holder").jsoneditor('validate');
});
```

### Get/Set Value

Set the editor's value
```js
$("#editor_holder").jsoneditor('value',{name: "John Smith"});
```

Get the editor's value
```js
var value = $("#editor_holder").jsoneditor('value');
console.log(value.name) // Will log "John Smith"
```

### Validate

When feasible, JSON Editor won't let users enter invalid data.
However, in some cases it is still possible to enter data that doesn't validate against the schema.

You can use the `validate` method to check if the data is valid or not.

```javascript
// Validate the editor's current value against the schema
var errors = $("#editor_holder").jsoneditor('validate');

if(errors.length) {
  // errors is an array of objects, each with a `path`, `property`, and `message` parameter
  // `property` is the schema keyword that triggered the validation error (e.g. "minLength")
  console.log(errors);
}
else {
  // It's valid!
}
```

By default, this will do the validation with the editor's current value.
If you want to use a different value, you can pass it in as a 2nd parameter.


```javascript
// Validate an arbitrary value against the editor's schema
var errors = $("#editor_holder").jsoneditor('validate',{
  value: {
    to: "test"
  }
});
```

### Listen for Changes

The `change` event is fired whenever the editor's value changes.  When using macro templates, multiple `change` events may fire in quick succession.

```javascript
$("#editor_holder").on('change',function() {});
```

### Destroy

This removes the editor HTML from the DOM and frees up memory.

```javascript
$("#editor_holder").jsoneditor('destroy');
```

CSS Integration
----------------
JSON Editor can integrate with several popular CSS frameworks out of the box.

The currently supported themes are:

*  html (the default)
*  bootstrap2
*  bootstrap3
*  foundation3
*  foundation4
*  foundation5
*  jqueryui

The default theme is `html`, which doesn't use any special class names or styling.
This default can be changed by setting the `$.jsoneditor.theme` variable.

```javascript
$.jsoneditor.theme = 'foundation5';
```

You can override this default on a per-instance basis by passing a `theme` parameter in when initializing:

```js
$("#editor_holder").jsoneditor({
  schema: schema,
  theme: 'jqueryui'
});
```

It's possible to create your own custom themes as well.  Look at any of the existing theme classes for examples.


JSON Schema Support
-----------------

JSON Editor fully supports version 3 and 4 of the JSON Schema [core][core] and [validation][validation] specifications.  The [hyper-schema][hyper] specification is not supported.

[core]: http://json-schema.org/latest/json-schema-core.html
[validation]: http://json-schema.org/latest/json-schema-validation.html
[hyper]: http://json-schema.org/latest/json-schema-hypermedia.html

### $ref and definitions

JSON Editor supports schema references to external urls and local definitions.  Here's an example showing both:

```json
{
  "type": "object",
  "properties": {
    "name": {
      "title": "Full Name",
      "$ref": "#/definitions/name"
    },
    "location": {
      "$ref": "http://mydomain.com/geo.json"
    }
  },
  "definitions": {
    "name": {
      "type": "string",
      "minLength": 5
    }
  }
}
```

Local references must point to the `definitions` object of the root node of the schema and can't be nested.
So, both `#/customkey/name` and `#/definitions/name/first` will throw an exception.

If loading an external url via Ajax, the url must either be on the same domain or return the correct HTTP cross domain headers.
If your urls don't meet this requirement, you can pass in the references to JSON Editor during initialization (see Usage section above).

### format

JSON Editor supports the following values for the `format` parameter for schemas of type `string`.  They will work with schemas of type `integer` and `number` as well, but some formats may produce weird results.
If the `enum` property is specified, `format` will be ignored.

*  bbcode (requires SCEditor)
*  color
*  date
*  datetime
*  datetime-local
*  email
*  hidden
*  html (requires SCEditor)
*  markdown (requires EpicEditor)
*  month
*  number
*  range
*  tel
*  text
*  textarea
*  time
*  url
*  week

JSON Editor uses HTML5 input types, so some of these may render as basic text input in older browsers.

Here is an example that will show a color picker in browsers that support it:

```json
{
  "type": "object",
  "properties": {
    "color": {
      "type": "string",
      "format": "color"
    }
  }
}
```

For schemas of type `array`, the format `table` is supported, which is a more compact UI for editing arrays.
To use this format, each array element must have the same schema.

```json
{
  "type": "array",
  "format": "table",
  "items": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string"
      }
    }
  }
}
```

Editor Options
----------------

Editors can accept options which alter the behavior in some way.

Right now, there is only 1 supported option

*  `collapsed` - If set to true for the `object`, `array`, or `table` editor, child editors will be collapsed by default.

```json
{
  "type": "object",
  "options": {
    "collapsed": true
  },
  "properties": {
    "name": {
      "type": "string" 
    }
  }
}
```

Templates and Variables
------------------
A unique feature of JSON Editor is the support for template macros.  This lets you specify a field's value in terms of other fields.  
Templates only work for fields of type `string`, `integer`, and `number`.

JSON Editor uses a barebones template engine by default (simple `{{variable}}` replacement only).

You can change the default by setting `$.jsoneditor.template` to one of the following supported template engines:

*  ejs
*  handlebars
*  hogan
*  markup
*  mustache
*  swig
*  underscore

```javascript
$.jsoneditor.template = 'handlebars';
```

You can set the template engine on a per-instance basis as well:

```js
$("#editor_holder").jsoneditor({
  schema: schema,
  template: 'hogan'
});
```

Here's an example template macro that generates an email address based on a first and last name:

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "object",
      "properties": {
        "first": {
          "type": "string"
        },
        "last": {
          "type": "string"
        }
      }
    },
    "email": {
      "title": "Generated Email",
      "type": "string",
      "template": "{{ fname }}.{{ lname }}@domain.com",
      "watch": {
        "fname": "name.first",
        "lname": "name.last"
      }
    }
  }
}
```

The `email` field will watch the `fname` and `lname` fields and re-calculate its value when either one changes.

Only fields specified in the `watch` object will be passed into the template.  By default, the paths (`name.first` and `name.last` in this example) are relative to the root schema.
You can make the variable paths relative to any ancestor node with a schema `id` defined as well.  This is especially useful within arrays.  Here's an example:

```json
{
  "type": "array",
  "items": {
    "id": "http://example.com/person",
    "type": "object",
    "properties": {
      "address": {
        "type": "object",
        "properties": {
          "city": {
            "type": "string"
          },
          "state": {
            "type": "string"
          }
        }
      },
      "location": {
        "type": "string",
        "template": "{{city}}, {{state}}",
        "watch": {
          "city": ["http://example.com/person","address.city"],
          "state": ["http://example.com/person","address.state"]
        }
      }
    }
  }
}
```

In this example, the `location` field in row X will watch row X's `address.city` and `address.state` fields.

### Custom Template Engines

If one of the included template engines isn't sufficient, 
you can use any custom template engine with a `compile` method.  For example:

```js
var myengine = {
  compile: function(template) {
    // Compile should return a render function
    return function(vars) {
      // A real template engine would render the template here
      var result = template;
      return result;
    }
  }
};

// Set globally
$.jsoneditor.template = myengine;

// Set on a per-instance basis
$("#editor").jsoneditor({
  schema: schema,
  template: myengine
});
```


Custom Editor Interfaces
-----------------

JSON Editor contains editor interfaces for each of the primitive JSON types as well as a few other specialized ones.

You can add custom editors interfaces fairly easily.  Look at any of the existing ones for an example.

JSON Editor uses resolver functions to determine which editor interface to use for a particular schema or subschema.

Let's say you make a custom `location` editor for editing geo data.  You can add a resolver function to use this custom editor when appropriate. For example:

```js
// Add a resolver function to the beginning of the resolver list
// This will make it run before any other ones
$.jsoneditor.resolvers.unshift(function(schema) {
  if(schema.type === "object" && schema.format === "location") {
    return "location";
  }
  
  // If no valid editor is returned, the next resolver function will be used
});
```

The following schema will now use this custom editor for each of the array elements instead of the default `object` editor.

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "format": "location",
    "properties": {
      "longitude": {
        "type": "number"
      },
      "latitude": {
        "type": "number"
      }
    }
  }
}
```

If you create a custom editor interface that you think could be helpful to others, submit a pull request!

The possibilities are endless.  Some ideas:

*  Syntax highlighting code editor (Ace, Markup, etc.)
*  A compact way to edit objects
*  Radio button version of the `select` editor
*  Typeahead for schemas with really long `enum` lists
*  Canvas based image editor that produces Base64 data urls


Custom Validation
----------------

JSON Editor provides a hook into the validation engine for adding your own custom validation.

Let's say you want to force all schemas with `format` set to `date` to match the pattern `YYYY-MM-DD`.

```js
// Custom validators must return an array of errors or an empty array if valid
$.jsoneditor.custom_validators.push(function(schema, value, path) {
  var errors = [];
  if(schema.format==="date") {
    if(!/^[0-9]{4}-[0-9]{2}-[0-9]{2}$/.test(value)) {
      // Errors must be an object with `path`, `property`, and `mesage`
      errors.push({
        path: path,
        property: 'format',
        message: 'Dates must be in the format "YYYY-MM-DD"'
      });
    }
  }
  return errors;
});
```
