# simple-schema

SimpleSchema validates JavaScript objects to ensure they match a schema. It can also clean the objects to automatically convert types, remove unsupported properties, and add automatic values such that the object is then more likely to pass validation.

There are a lot of similar packages for validating objects. These are some of the features of this package that might be good reasons to choose this one over another:
- Isomorphic. Works in NodeJS and modern browsers.
- The object you validate can be a MongoDB modifier. SimpleSchema understands how to properly validate it such that the object in the database, after undergoing modification, will be valid.
- Optional Tracker reactivity for Meteor apps
- Powerful customizable error message system with decent English language defaults and support for localization, which makes it easy to drop this package in and display the validation error messages to end users.
- Recommended by MDG, the group that makes Meteor
- Has hundreds of tests and is used in production apps of various sizes
- Used by the [Collection2](https://github.com/aldeed/meteor-collection2) and [AutoForm](https://github.com/aldeed/meteor-autoform) Meteor packages.

There are also reasons not to choose this package. Because of all it does, this package is more complex than (but still "simple" :) ) and slower than some other packages. Based on your needs, you should decide whether these tradeoffs are acceptable. One faster but less powerful option is [simplecheck](https://www.npmjs.com/package/simplecheck).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [simple-schema](#simple-schema)
  - [The History of SimpleSchema](#the-history-of-simpleschema)
  - [Installation](#installation)
  - [Lingo](#lingo)
  - [Quick Start](#quick-start)
    - [Validate an Object and Throw an Error](#validate-an-object-and-throw-an-error)
    - [Validate an Array of Objects and Throw an Error](#validate-an-array-of-objects-and-throw-an-error)
    - [Validate an Object and Get the Errors](#validate-an-object-and-get-the-errors)
    - [Validate a MongoDB Modifier](#validate-a-mongodb-modifier)
    - [Enable Meteor Tracker Reactivity](#enable-meteor-tracker-reactivity)
    - [Automatically Clean the Object Before Validating It](#automatically-clean-the-object-before-validating-it)
    - [Explicitly Clean an Object](#explicitly-clean-an-object)
  - [Defining a Schema](#defining-a-schema)
    - [Shorthand Definitions](#shorthand-definitions)
    - [Longhand Definitions](#longhand-definitions)
    - [Mixing Shorthand with Longhand](#mixing-shorthand-with-longhand)
    - [More Shorthand](#more-shorthand)
    - [Multiple Definitions For One Key](#multiple-definitions-for-one-key)
    - [Extending Schemas](#extending-schemas)
      - [Overriding When Extending](#overriding-when-extending)
    - [Subschemas](#subschemas)
    - [Extracting Schemas](#extracting-schemas)
  - [Schema Keys](#schema-keys)
  - [Schema Rules](#schema-rules)
    - [type](#type)
    - [label](#label)
    - [optional](#optional)
    - [min/max](#minmax)
    - [exclusiveMin/exclusiveMax](#exclusiveminexclusivemax)
    - [minCount/maxCount](#mincountmaxcount)
    - [allowedValues](#allowedvalues)
    - [regEx](#regex)
    - [blackbox](#blackbox)
    - [trim](#trim)
    - [custom](#custom)
    - [defaultValue](#defaultvalue)
    - [autoValue](#autovalue)
  - [Validating Data](#validating-data)
    - [The Object to Validate](#the-object-to-validate)
    - [Ways to Perform Validation](#ways-to-perform-validation)
      - [Named Validation Contexts](#named-validation-contexts)
      - [Unnamed Validation Contexts](#unnamed-validation-contexts)
    - [Validating an Object](#validating-an-object)
    - [Validating Only Some Keys in an Object](#validating-only-some-keys-in-an-object)
    - [Validation Options](#validation-options)
    - [Validating and Throwing ValidationErrors](#validating-and-throwing-validationerrors)
      - [Customize the Error That is Thrown](#customize-the-error-that-is-thrown)
    - [Custom Field Validation](#custom-field-validation)
    - [Custom Whole-Document Validators](#custom-whole-document-validators)
    - [Manually Adding a Validation Error](#manually-adding-a-validation-error)
    - [Asynchronous Custom Validation on the Client](#asynchronous-custom-validation-on-the-client)
    - [Getting a List of Invalid Keys and Validation Error Messages](#getting-a-list-of-invalid-keys-and-validation-error-messages)
  - [Customizing Validation Messages](#customizing-validation-messages)
  - [Other Validation Context Methods](#other-validation-context-methods)
  - [Other SimpleSchema Methods](#other-simpleschema-methods)
  - [Cleaning Objects](#cleaning-objects)
  - [Dates](#dates)
  - [Best Practice Code Examples](#best-practice-code-examples)
    - [Make a field conditionally required](#make-a-field-conditionally-required)
    - [Validate one key against another](#validate-one-key-against-another)
  - [Debug Mode](#debug-mode)
  - [Extending the Schema Options](#extending-the-schema-options)
  - [Add On Packages](#add-on-packages)
  - [License](#license)
  - [Contributing](#contributing)
    - [Thanks](#thanks)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## The History of SimpleSchema

SimpleSchema was first released as a Meteor package in mid-2013. Version 1.0 was released in September 2014. In mid-2016, version 2.0 was released as an NPM package, which can be used in Meteor, NodeJS, or static browser apps.

If you are migrating from the Meteor package, refer to the [CHANGELOG](https://github.com/aldeed/meteor-simple-schema/blob/v2/CHANGELOG.md#200)

## Installation

```
npm install simpl-schema
```

There are other NPM packages named `simpleschema` and `simple-schema`. Make sure you install the right package. There is no "e" on "simpl".

NOTE: You may also need to load the `babel-polyfill` package if you get any errors related to missing ES6 functions in certain browsers.

## Lingo

In this documentation:
- "key", "field", and "property" generally all mean the same thing: an identifier for some part of an object that is validated by your schema. SimpleSchema uses dot notation to identify nested keys.
- "validate" means to check whether an object matches what you expect, for example, having the expected keys with the expected data types, expected string lengths, etc.

## Quick Start

### Validate an Object and Throw an Error

```js
import SimpleSchema from 'simpl-schema';

new SimpleSchema({
  name: String,
}).validate({
  name: 2,
});
```

### Validate an Array of Objects and Throw an Error

An error is thrown for the first invalid object found.

```js
import SimpleSchema from 'simpl-schema';

new SimpleSchema({
  name: String,
}).validate([
  { name: 'Bill' },
  { name: 2 },
]);
```

### Validate an Object and Get the Errors

```js
import SimpleSchema from 'simpl-schema';

const validationContext = new SimpleSchema({
  name: String,
}).newContext();

validationContext.validate({
  name: 2,
});

console.log(validationContext.isValid());
console.log(validationContext.validationErrors());
```

### Validate a MongoDB Modifier

```js
import SimpleSchema from 'simpl-schema';

const validationContext = new SimpleSchema({
  name: String,
}).newContext();

validationContext.validate({
  $set: {
    name: 2,
  },
}, { modifier: true });

console.log(validationContext.isValid());
console.log(validationContext.validationErrors());
```

### Enable Meteor Tracker Reactivity

```js
import SimpleSchema from 'simpl-schema';
import { Tracker } from 'meteor/tracker';

const validationContext = new SimpleSchema({
  name: String,
}, { tracker: Tracker }).newContext();

Tracker.autorun(function () {
  console.log(validationContext.isValid());
  console.log(validationContext.validationErrors());
});

validationContext.validate({
  name: 2,
});

validationContext.validate({
  name: 'Joe',
});
```

### Automatically Clean the Object Before Validating It

TO DO

### Set Default Cleaning Options

```js
import SimpleSchema from 'simpl-schema';

const mySchema = new SimpleSchema({
  name: String,
}, {
  clean: {
    filter: true,
    autoConvert: true,
    removeEmptyStrings: true,
    trimStrings: true,
    getAutoValues: true,
    removeNullsFromArrays: true,
  },
});
```

### Explicitly Clean an Object

```js
import SimpleSchema from 'simpl-schema';

const mySchema = new SimpleSchema({ name: String });
const doc = { name: 123 };
const cleanDoc = mySchema.clean(doc);
// cleanDoc is now mutated to hopefully have a better chance of passing validation
console.log(typeof cleanDoc.name); // string
```

Works for a MongoDB modifier, too:

```js
import SimpleSchema from 'simpl-schema';

const mySchema = new SimpleSchema({ name: String });
const modifier = { $set: { name: 123 } };
const cleanModifier = mySchema.clean(modifier);
// doc is now mutated to hopefully have a better chance of passing validation
console.log(typeof cleanModifier.$set.name); // string
```

## Defining a Schema

Let's get into some more details about the different syntaxes that are supported when defining a schema. It's probably best to start with the simplest syntax. Here's an example:

### Shorthand Definitions

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  name: String,
  age: SimpleSchema.Integer,
  registered: Boolean,
});
```

This is referred to as "shorthand" syntax. You simply map a property name to a type. When validating, SimpleSchema will make sure that all of those properties are present and are set to a value of that type.

### Longhand Definitions

In many cases, you will need to use longhand in order to define additional rules beyond what the data type should be.

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  name: {
    type: String,
    max: 40,
  },
  age: {
    type: SimpleSchema.Integer,
    optional: true,
  },
  registered: {
    type: Boolean,
    defaultValue: false,
  },
});
```

### Mixing Shorthand with Longhand

You can use any combination of shorthand and longhand:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  name: String,
  age: {
    type: SimpleSchema.Integer,
    optional: true,
  },
  registered: Boolean,
});
```

### More Shorthand

If you set the schema key to a regular expression, then the `type` will be `String` and the string must match the provided regular expression.

For example, this:

```js
{
  exp: /foo/
}
```

is equivalent to:

```js
{
  exp: { type: String, regEx: /foo/ }
}
```

You can also set the schema key to an array of some type:

```js
{
  friends: [String],
}
```

is equivalent to:

```js
{
  friends: { type: Array },
  'friends.$': { type: String },
}
```

### Multiple Definitions For One Key

You can define two or more different ways in which a key will be considered valid:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  id: SimpleSchema.oneOf(String, SimpleSchema.Integer),
  name: String,
});
```

And this can be done in any mixture of shorthand and longhand:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  id: SimpleSchema.oneOf({
    type: String,
    min: 16,
    max: 16,
  }, {
    type: SimpleSchema.Integer,
    min: 0,
  }),
  name: String,
});
```

NOTE: Multiple definitions is still an experimental feature and may not work as you expect in complex situations, such as where one of the valid definitions is an object or array. By reporting any weirdness you experience, you can help make it more robust.

### Extending Schemas

If there are certain fields that are repeated in many of your schemas, it can be useful to define a SimpleSchema instance just for those fields and then merge them into other schemas:

```js
import SimpleSchema from 'simpl-schema';
import { idSchema, addressSchema } from './sharedSchemas';

const schema = new SimpleSchema({
  name: String,
});
schema.extend(idSchema);
schema.extend(addressSchema);
```

#### Overriding When Extending

If the key appears in both schemas, the definition will be extended such that the result is the combination of both definitions.

```js
import SimpleSchema from 'simpl-schema';
import { idSchema, addressSchema } from './sharedSchemas';

const schema = new SimpleSchema({
  name: {
    type: String,
    min: 5,
  },
});
schema.extend({
  name: {
    type: String,
    max: 15,
  },
});
```

The above will result in the definition of the `name` field becoming:

```js
{
  name: {
    type: String,
    min: 5,
    max: 15,
  },
}
```

Note also that a plain object was passed to `extend`. If you pass a plain object, it is converted to a `SimpleSchema` instance for you.

### Subschemas

Similar to extending, you can also reference other schemas as a way to define objects that occur within the main object:

```js
import SimpleSchema from 'simpl-schema';
import { addressSchema } from './sharedSchemas';

const schema = new SimpleSchema({
  name: String,
  homeAddress: addressSchema,
  billingAddress: {
    type: addressSchema,
    optional: true,
  },
});
```

### Extracting Schemas

Sometimes you have one large SimpleSchema object, and you need just a subset of it for some purpose. To pull out certain schema keys into a new schema, you can use the `pick` method:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  firstName: String,
  lastName: String,
  username: String,
});

const nameSchema = schema.pick('firstName', 'lastName');
```

## Schema Keys

A basic schema key is just the name of the key (property) to expect in the objects that will be validated.

Use string keys with MongoDB-style dot notation to validate nested arrays and objects. For example:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  mailingAddress: Object,
  'mailingAddress.street': String,
  'mailingAddress.city': String,
});
```

To indicate array items, use a `$`:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  addresses: {
    type: Array,
    minCount: 1,
    maxCount: 4
  },
  'addresses.$': Object,
  'addresses.$.street': String,
  'addresses.$.city': String,
});
```

## Schema Rules

Here are some specifics about the various rules you can define in your schema.

### type

One of the following:

* `String`
* `Number`
* `SimpleSchema.Integer` (same as `Number` but with decimals/floats disallowed)
* `Boolean`
* `Object`
* `Array`
* Any custom or built-in class like `Date`
* Another `SimpleSchema` instance, meaning `Object` type with this schema
* `SimpleSchema.oneOf(...)`, with multiple of the above types

### label

*Can also be a function that returns the label*

A string that will be used to refer to this field in validation error messages. The default is an inflected (humanized) derivation of the key name itself. For example, the key "firstName" will have a default label of "First name" if you do not include the `label` property in your definition.

You can use the `labels` function to alter one or more labels on the fly:

```js
schema.labels({
  password: "Enter your password"
});
```

If you have enabled Tracker reactivity, this method causes reactive labels to update.

To get the label for a field, use `schema.label(fieldName)`, which returns a usable string. If you have enabled Tracker reactivity, this method is reactive.

### optional

*Can also be a function that returns true or false*

By default, all keys are required. Set `optional: true` to change that.

With complex keys, it might be difficult to understand what "required" means. Here's a brief explanation of how requiredness is interpreted:

* If `type` is `Array`, then "required" means that key must have a value, but an empty array is fine. (If an empty array is *not* fine, add the `minCount: 1` option.)
* For array items (when the key name ends with ".$"), the `optional` option has no effect. That is, something cannot be "required" to be in an array.
* If a key is required at a deeper level, the key must have a value *only if* the object it belongs to is present.
* When the object being validated is a Mongo modifier object, changes that would unset or `null` a required key result in validation errors.

That last point can be confusing, so let's look at a couple examples:

* Say you have a required key "friends.address.city" but "friends.address" is optional. If "friends.address" is set in the object you're validating, but "friends.address.city" is not, there is a validation error. However, if "friends.address" is *not* set, then there is no validation error for "friends.address.city" because the object it belongs to is not present.
* If you have a required key "friends.$.name", but the `friends` array has no objects in the object you are validating, there is no validation error for "friends.$.name". When the `friends` array *does* have objects, every present object is validated, and each object could potentially have a validation error if it is missing the `name` property. For example, when there are two objects in the friends array and both are missing the `name` property, there will be a validation error for both "friends.0.name" and "friends.1.name".

### min/max

*Can also be a function that returns the min/max value*

* If `type` is `Number` or `SimpleSchema.Integer`, these rules define the minimum or maximum numeric value.
* If `type` is `String`, these rules define the minimum or maximum string length.
* If `type` is `Date`, these rules define the minimum or maximum date, inclusive.

You can alternatively provide a function that takes no arguments and returns the appropriate minimum or maximum value. This is useful, for example, if the minimum Date for a field should be "today".

### exclusiveMin/exclusiveMax

*Can also be a function that returns true or false*

Set to `true` to indicate that the range of numeric values, as set by min/max, are to be treated as an exclusive range. Set to `false` (default) to treat ranges as inclusive.

### minCount/maxCount

*Can also be a function that returns the minCount/maxCount value*

Define the minimum or maximum array length. Used only when type is `Array`.

### allowedValues

*Can also be a function that returns the array of allowed values*

An array of values that are allowed. A key will be invalid if its value is not one of these.

You can use `schema.getAllowedValuesForKey(key)` to get the allowed values array for a key.

### regEx

*Can also be a function that returns a regular expression or an array of them*

Any regular expression that must be matched for the key to be valid, or an array of regular expressions that will be tested in order.

The `SimpleSchema.RegEx` object defines standard regular expressions you can use as the value for the `regEx` key.
- `SimpleSchema.RegEx.Email` for emails (uses a permissive regEx recommended by W3C, which most browsers use. Does not require a TLD)
- `SimpleSchema.RegEx.EmailWithTLD` for emails that must have the TLD portion (.com, etc.). Emails like `me@localhost` and `me@192.168.1.1` won't pass this one.
- `SimpleSchema.RegEx.Domain` for external domains and the domain only (requires a tld like `.com`)
- `SimpleSchema.RegEx.WeakDomain` for less strict domains and IPv4 and IPv6
- `SimpleSchema.RegEx.IP` for IPv4 or IPv6
- `SimpleSchema.RegEx.IPv4` for just IPv4
- `SimpleSchema.RegEx.IPv6` for just IPv6
- `SimpleSchema.RegEx.Url` for http, https and ftp urls
- `SimpleSchema.RegEx.Id` for IDs generated by `Random.id()` of the random package, also usable to validate a relation id.
- `SimpleSchema.RegEx.ZipCode` for 5- and 9-digit ZIP codes
- `SimpleSchema.RegEx.Phone` for phone numbers (taken from Google's libphonenumber library)

### blackbox

If you have a key with type `Object`, the properties of the object will be validated as well, so you must define all allowed properties in the schema. If this is not possible or you don't care to validate the object's properties, use the `blackbox: true` option to skip validation for everything within the object.

Prior to SimpleSchema 2.0, objects that are instances of a custom class were considered to be blackbox by default. This is no longer true, so if you do not want your class instance validated, be sure to add `blackbox: true` in your schema.

### trim

*Used by the cleaning process but not by validation*

When you call `simpleSchemaInstance.clean()` with `trimStrings` set to `true`, all string values are trimmed of leading and trailing whitespace. If you set `trim` to `false` for certain keys in their schema definition, those keys will be skipped.

### custom

Refer to the [Custom Validation](#custom-validation) section.

### defaultValue

*Used by the cleaning process but not by validation*

Set this to any value that you want to be used as the default when an object does not include this field or has this field set to `undefined`. This value will be injected into the object by a call to `mySimpleSchema.clean()` with `getAutovalues: true`. Default values are set only when cleaning *non-modifier* objects.

Note the following points of confusion:

* A default value itself is not cleaned. So, for example, if your default value is "", it will not be removed by the `removeEmptyStrings` operation in the cleaning.
* A default value is *always* added if there isn't a value set. Even if the property is a child of an optional object, and the optional object is not present, the object will be added and its property will be set to the default value. Effectively, this means that if you provide a default value for one property of an object, you must provide a default value for all properties of that object or risk confusing validation errors.

If you need more control, use the `autoValue` option instead.

### autoValue

*Used by the cleaning process but not by validation*

The `autoValue` option allows you to specify a function that is called by `simpleSchemaInstance.clean()` to potentially change the value of a property in the object being cleaned. This is a powerful feature that allows you to set up either forced values or default values, potentially based on the values of other fields in the object.

An `autoValue` function is passed the document or modifier as its only argument, but you will generally not need it. Instead, the function context provides a variety of properties and methods to help you determine what you should return.

If an `autoValue` function does not return anything (i.e., returns `undefined`), the field's value will be whatever the document or modifier says it should be. If that field is already in the document or modifier, it stays in the document or modifier with the same value. If it's not in the document or modifier, it's still not there. If you don't want it to be in the doc or modifier, you must call `this.unset()`.

Any other return value will be used as the field's value. You may also return special pseudo-modifier objects for update operations. Examples are `{$inc: 1}` and `{$push: new Date}`.

The following properties and methods are available in `this` for an `autoValue` function:

* `isSet`: True if the field is already set in the document or modifier
* `unset()`: Call this method to prevent the original value from being used when you return undefined.
* `value`: If isSet = true, this contains the field's current (requested) value in the document or modifier.
* `operator`: If isSet = true and isUpdate = true, this contains the name of the update operator in the modifier in which this field is being changed. For example, if the modifier were `{$set: {name: "Alice"}}`, in the autoValue function for the `name` field, `this.isSet` would be true, `this.value` would be "Alice", and `this.operator` would be "$set".
* `field()`: Use this method to get information about other fields. Pass a field name (schema key) as the only argument. The return object will have `isSet`, `value`, and `operator` properties for that field.
* `siblingField()`: Use this method to get information about other fields that have the same parent object. Works the same way as `field()`. This is helpful when you use sub-schemas or when you're dealing with arrays of objects.

## Validating Data

### The Object to Validate

The object you pass in when validating can be a normal object, or it can be
a Mongo modifier object (with `$set`, etc. keys). In other words, you can pass
in the exact object that you are going to pass to `Collection.insert()` or
`Collection.update()`. This is what the [collection2](https://atmospherejs.com/aldeed/collection2) package does for you.

### Ways to Perform Validation

There are three ways to validate an object against your schema:

1. With a throwaway context, throwing an Error for the first validation error found (schema.validate())
2. With a unique unnamed validation context, not throwing any Errors (schema.newContext().validate())
3. With a unique named validation context, not throwing any Errors (schema.namedContext('someUniqueString').validate())
4. With the default validation context, not throwing any Errors. (schema.namedContext().validate())

A validation context provides reactive methods for validating and checking the validation status of a particular object.

#### Named Validation Contexts

It's usually best to use a named validation context. That way, the context is automatically persisted by name, allowing you to easily rely on its reactive methods.

Here is an example of obtaining a named validation context:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  name: String,
});

const userFormValidationContext = schema.namedContext('userForm');
```

The first time you request a context with a certain name, it is created. Calling `namedContext()` passing no arguments is equivalent to calling `namedContext('default')`.

#### Unnamed Validation Contexts

To obtain an unnamed validation context, call `newContext()`:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  name: String,
});

const myValidationContext = schema.newContext();
```

An unnamed validation context is not persisted anywhere. It can be useful when you need to see if a document is valid but you don't need any of the reactive methods for that context, or if you are going to keep the context reference in memory yourself.

### Validating an Object

To validate an object against the schema in a validation context, call `validationContextInstance.validate(obj, options)`. This method returns `true` if the object is valid according to the schema or `false` if it is not. It also stores a list of invalid fields and corresponding error messages in the context object and causes the reactive methods to react if you injected Tracker reactivity.

You can call `myContext.isValid()` to see if the object last passed into `validate()` was found to be valid. This is a reactive method that returns `true` or `false`.

For a list of options, see the [Validation Options](#validation-options) section.

### Validating Only Some Keys in an Object

You may have the need to (re)validate certain keys while leaving any errors for other keys unchanged. For example, if you have several errors on a form and you want to revalidate only the invalid field the user is currently typing in. For this situation, call `myContext.validate` with the `keys` option set to an array of keys that should be validated. This may cause all of the reactive methods to react.

This method returns `true` only if all the specified schema keys and their descendent keys are valid according to the schema. Otherwise it returns `false`.

### Validation Options

`validate()` accepts the following options:

* `modifier`: Are you validating a Mongo modifier object? False by default.
* `upsert`: Are you validating a Mongo modifier object potentially containing upsert operators? False by default.
* `extendedCustomContext`: This object will be added to the `this` context in any custom validation functions that are run during validation. See the [Custom Validation](#custom-validation) section.
* `ignore`: An array of validation error types (in SimpleSchema.ErrorTypes enum) to ignore.
* `keys`: An array of keys to validate. If not provided, revalidates the entire object.

### Validating and Throwing ValidationErrors

- Call `mySimpleSchema.validate(obj, options)` to validate `obj` against the schema and throw a `ValidationError` if invalid.
- Call `SimpleSchema.validate(obj, schema, options)` static function as a shortcut for `mySimpleSchema.validate` if you don't want to create `mySimpleSchema` first. The `schema` argument can be just the schema object, in which case it will be passed to the `SimpleSchema` constructor for you. This is like `check(obj, schema)` but without the `check` dependency and with the ability to pass full schema error details back to a callback on the client.
- Call `mySimpleSchema.validator()` to get a function that calls `mySimpleSchema.validate` for whatever object is passed to it. This means you can do `validate: mySimpleSchema.validator()` in the [mdg:validated-method](https://github.com/meteor/validated-method) package.

#### Customize the Error That is Thrown

```js
import SimpleSchema from 'simpl-schema';

SimpleSchema.defineValidationErrorTransform(error => {
  const customError = new MyCustomErrorType(error.message);
  customError.errorList = error.details;
  return customError;
});
```

### Custom Field Validation

There are three ways to attach custom validation methods.

To add a custom validation function that is called for ALL keys in ALL schemas (for example, to publish a package that adds global support for some additional rule):

```js
SimpleSchema.addValidator(myFunction);
```

To add a custom validation function that is called for ALL keys for ONE schema:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({ ... });
schema.addValidator(myFunction);
```

To add a custom validation function that is called for ONE key in ONE schema:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  someKey: {
    type: String,
    custom: myFunction,
  }
});
```

All custom validation functions work the same way. First, do the necessary custom validation, use `this` to get whatever information you need. Then, if valid, return `undefined`. If invalid, return an error type string. The error type string can be one of the [built-in strings](#manually-adding-a-validation-error) or any string you want.
* If you return a built-in string, it's best to use the `SimpleSchema.ErrorTypes` constants.
* If you return a custom string, you'll usually want to [define a message for it](#customizing-validation-messages).

Within your custom validation function, `this` provides the following properties:
* `key`: The name of the schema key (e.g., "addresses.0.street")
* `genericKey`: The generic name of the schema key (e.g., "addresses.$.street")
* `definition`: The schema definition object.
* `isSet`: Does the object being validated have this key set?
* `value`: The value to validate.
* `operator`: The Mongo operator for which we're doing validation. Might be `null`.
* `validationContext`: The current `ValidationContext` instance
* `field()`: Use this method to get information about other fields. Pass a field name (non-generic schema key) as the only argument. The return object will have `isSet`, `value`, and `operator` properties for that field.
* `siblingField()`: Use this method to get information about other fields that have the same parent object. Works the same way as `field()`. This is helpful when you use sub-schemas or when you're dealing with arrays of objects.
* `addValidationErrors(errors)`: Call this to add validation errors for any key. In general, you should use this to add errors for other keys. To add an error for the current key, return the error type string. If you do use this to add an error for the current key, return `false` from your custom validation function.

NOTE: If you need to do some custom validation on the server and then display errors back
on the client, refer to the [Asynchronous Custom Validation on the Client](#asynchronous-custom-validation-on-the-client) section.

### Custom Whole-Document Validators

Add a validator for all schemas:

```js
import SimpleSchema from 'simpl-schema';

SimpleSchema.addDocValidator(obj => {
  // Must return an array, potentially empty, of objects with `name` and `type` string properties and optional `value` property.
  return [
    { name: 'firstName', type: 'TOO_SILLY', value: 'Reepicheep' }
  ];
});
```

Add a validator for one schema:

```js
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({ ... });
schema.addDocValidator(obj => {
  // Must return an array, potentially empty, of objects with `name` and `type` string properties and optional `value` property.
  return [
    { name: 'firstName', type: 'TOO_SILLY', value: 'Reepicheep' }
  ];
});
```

### Manually Adding a Validation Error

If you want to reactively display an arbitrary validation error and it is not possible to use a custom validation function (perhaps you have to call a function `onSubmit` or wait for asynchronous results), you can add one or more errors to a validation context at any time by calling `myContext.addValidationErrors(errors)`, where `errors` is an array of error objects with the following format:

```js
{name: key, type: errorType, value: anyValue}
```

* `name`: The schema key as specified in the schema.
* `type`: The type of error. Any string you want, or one of the strings in the `SimpleSchema.ErrorTypes` list.
* `value`: Optional. The value that was not valid. Will be used to replace the
`[value]` placeholder in error messages.

If you use a custom string for `type`, be sure to define a message for it. (See [Customizing Validation Messages](#customizing-validation-messages)).

Example:

```js
SimpleSchema.messageBox.messages({
    en: {
        wrongPassword: "Wrong password"
    }
});

myValidationContext.addValidationErrors([{name: "password", type: "wrongPassword"}]);
```

### Asynchronous Custom Validation on the Client

Validation runs synchronously for many reasons, and likely always will. This makes it difficult to wait for asynchronous results as part of custom validation. Here's one example of how you might validate that a username is unique on the client, without publishing all usernames to every client:

```js
username: {
  type: String,
  regEx: /^[a-z0-9A-Z_]{3,15}$/,
  unique: true,
  custom() {
    if (Meteor.isClient && this.isSet) {
      Meteor.call("accountsIsUsernameAvailable", this.value, (error, result) => {
        if (!result) {
          this.validationContext.addValidationErrors([{
            name: "username",
            type: "notUnique"
          }]);
        }
      });
    }
  }
}
```

Note that we're calling our "accountsIsUsernameAvailable" server method and waiting for an asynchronous result, which is a boolean that indicates whether that username is available. If it's taken, we manually invalidate the `username` key with a "notUnique" error.

This doesn't change the fact that validation is synchronous. If you use this with an autoform and there are no validation errors, the form would still be submitted. However, the user creation would fail and a second or two later, the form would display the "notUnique" error, so the end result is very similar to actual asynchronous validation.

You can use a technique similar to this to work around asynchronicity issues in both client and server code.

### Getting a List of Invalid Keys and Validation Error Messages

_This is a reactive method if you have enabled Tracker reactivity._

Call `myValidationContext.validationErrors()` to get the full array of validation errors. Each object in the array has at least two keys:
* `name`: The schema key as specified in the schema.
* `type`: The type of error. See `SimpleSchema.ErrorTypes`.

There may also be a `value` property, which is the value that was invalid.

There may be a `message` property, but usually the error message is constructed from message templates. You should call `ctxt.keyErrorMessage(key)` to get a reactive message string rather than using `error.message` directly.

## Customizing Validation Messages

Error messages are managed by the [message-box](https://github.com/aldeed/node-message-box) package. You can set global defaults using `MessageBox.defaults`, and you can access the instance used by a schema at `simpleSchemaInstance.messageBox`;

## Other Validation Context Methods

`myContext.keyIsInvalid(key)` returns true if the specified key is currently
invalid, or false if it is valid. This is a reactive method.

`myContext.keyErrorMessage(key)` returns the error message for the specified
key if it is invalid. If it is valid, this method returns an empty string. This
is a reactive method.

Call `myContext.reset()` if you need to reset the validation context, clearing out any invalid field messages and making it valid.

## Other SimpleSchema Methods

Call `MySchema.schema([key])` to get the schema definition object. If you specify a key, then only the schema definition for that key is returned.

Note that this may not match exactly what you passed into the SimpleSchema constructor. The schema definition object is normalized internally, and this method returns the normalized copy.

## Cleaning Objects

You can call `simpleSchemaInstance.clean()` or `simpleSchemaValidationContextInstance.clean()` to clean the object you're validating. Do this prior to validating it to avoid any avoidable validation errors.

The `clean` function takes the object to be cleaned as its first argument and the following optional options as its second argument:

* `mutate`: The object is copied before being cleaned. If you don't mind mutating the object you are cleaning, you can pass `mutate: true` to get better performance.
* `isModifier`: Is the first argument a modifier object? False by default.
* `filter`: `true` by default. If `true`, removes any keys not explicitly or implicitly allowed by the schema, which prevents errors being thrown for those keys during validation.
* `autoConvert`: `true` by default. If `true`, helps eliminate unnecessary validation messages by automatically converting values where possible.
  * Non-string values are converted to a String if the schema expects a String
  * Strings that are numbers are converted to Numbers if the schema expects a Number
  * Non-array values are converted to a one-item array if the schema expects an Array
* `removeEmptyStrings`: Remove keys in normal object or $set where the value is an empty string? True by default.
* `trimStrings`: Remove all leading and trailing spaces from string values? True by default.
* `getAutoValues`: Run `autoValue` functions and inject automatic and `defaultValue` values? True by default.
* `extendAutoValueContext`: This object will be added to the `this` context of autoValue functions. `extendAutoValueContext` can be used to give your `autoValue` functions additional valuable information, such as `userId`. (Note that operations done using the Collection2 package automatically add `userId` to the `autoValue` context already.)

You can also set defaults for any of these options in your SimpleSchema constructor options:

```js
const schema = new SimpleSchema({
  name: String
}, {
  clean: {
    trimStrings: false,
  },
});
```

NOTE: The Collection2 package always calls `clean` before every insert, update, or upsert.

## Dates

For consistency, if you care only about the date (year, month, date) portion and not the time, then use a `Date` object set to the desired date at midnight UTC. This goes for `min` and `max` dates, too. If you care only about the date
portion and you want to specify a minimum date, `min` should be set to midnight UTC on the minimum date (inclusive).

Following these rules ensures maximum interoperability with HTML5 date inputs and usually just makes sense.

## Best Practice Code Examples

### Make a field conditionally required

If you have a field that should be required only in certain circumstances, first make the field
optional, and then use a custom function similar to this:

```js
{
  field: {
    type: String,
    optional: true,
    custom: function () {
      let shouldBeRequired = this.field('saleType').value === 1;

      if (shouldBeRequired) {
        // inserts
        if (!this.operator) {
          if (!this.isSet || this.value === null || this.value === "") return SimpleSchema.ErrorTypes.REQUIRED;
        }

        // updates
        else if (this.isSet) {
          if (this.operator === "$set" && this.value === null || this.value === "") return SimpleSchema.ErrorTypes.REQUIRED;
          if (this.operator === "$unset") return SimpleSchema.ErrorTypes.REQUIRED;
          if (this.operator === "$rename") return SimpleSchema.ErrorTypes.REQUIRED;
        }
      }
    }
  }
}
```

Where `customCondition` is whatever should trigger it being required.

Note: In the future we could make this a bit simpler by allowing `optional` to be a function that returns
true or false. Pull request welcome.

### Validate one key against another

Here's an example of declaring one value valid or invalid based on another
value using a custom validation function.

```js
SimpleSchema.messages({
  "passwordMismatch": "Passwords do not match"
});

MySchema = new SimpleSchema({
  password: {
    type: String,
    label: "Enter a password",
    min: 8
  },
  confirmPassword: {
    type: String,
    label: "Enter the password again",
    min: 8,
    custom: function () {
      if (this.value !== this.field('password').value) {
        return "passwordMismatch";
      }
    }
  }
});
```

## Debug Mode

Set `SimpleSchema.debug = true` in your app before creating any named
validation contexts to cause all named validation contexts to automatically
log all invalid key errors to the browser console. This can be helpful while
developing an app to figure out why certain actions are failing validation.

## Extending the Schema Options

You may find at some point that there is something extra you would really like to define within a schema for your package or app. However, if you add unrecognized options to your schema definition, you will get an error. To inform SimpleSchema about your custom option and avoid the error, you need to call `SimpleSchema.extendOptions`. By way of example, here is how the Collection2 package adds the additional schema options it provides:

```js
SimpleSchema.extendOptions(['index', 'unique', 'denyInsert', 'denyUpdate']);
```

Obviously you need to ensure that `extendOptions` is called before any SimpleSchema instances are created with those options.

## Add On Packages

[mxab:simple-schema-jsdoc](https://atmospherejs.com/mxab/simple-schema-jsdoc) Generate jsdoc from your schemas.

(Submit a PR to list your package here)

## License

MIT

## Contributing

Anyone is welcome to contribute. Before submitting a pull request, make sure that you've added tests for your changes, and that all tests pass when you run `npm test`.

### Thanks

(Add your name if it's missing.)

- @mquandalle
- @Nemo64
