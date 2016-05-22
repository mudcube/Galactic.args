# Galactic.args(...)

#### The easy way to handle arguments! No dependancies. Vanilla JS.
 
 *... or "The guide to Nonviolent Communications for Javascript Arguments Handling" ;)*

### Index
* [What is a schema?](#what-is-a-schema)
* [Schema pattern arguments](#schema-pattern-arguments)
* [Shorthand schema example](#schema-shorthand-patterns)
* [Longhand schema example](#schema-longhand-patterns-are-more-flexible)
* [Combining schema patterns](#multiple-schema-patterns-can-be-used-in-combination)
* [Nested validation of Objects](#nested-validation-on-objects-looks-like-this)
* [Nested validation of Arrays](#nested-validation-on-arrays-looks-like-this)

### Basic Usage

***
#### `Galactic.args(...)` creates a validator based on your schema:

```js
	let validator = Galactic.args(schema, onInvalid)
```

***

#### The validator has 2-methods:

* **.parse(input, onInvalid)** - Returns an object with data consistency based on you schema.
	* `input` can be an Object, Array, or the arguments object.
	* `onInvalid` is the function that fires if `input` is invalid.

* **.wrap(handler, onInvalid)** - Returns a wrapped version of your handler function that ensures data consistency while maintaining your 'handler's `this` scope.
	* `handler` is the function you want wrapped. Successful method calls are directed here.
	* `onInvalid` is the function that fires if data sent into the wrapper is invalid.

***

#### Here's a basic example of wrapping a function using shorthand patterns:

```js
	let yourFunction = Galactic.args({
		yourInt: 'number=0', // is a number with a 'defaultValue' of 0
		yourString: 'string?', // is an optional string
		yourValue: 'number|boolean' // is a number or a boolean
	}).wrap(function (args) { // this is your handler
		console.log(args) // logs Object { yourInt, yourString, yourValue }
	}, function (e) {
		console.log(e.errors) // logs details on why the values failed validation
		console.log(e.failed) // logs values that failed validation
		console.log(e.passed) // logs values that passed validation
	})
```

***

#### Now you can call `yourFunction(...)` in multiple ways with the same results!

```js
	/* can be called with arguments */
	yourFunction(42, 'Hi!', true)
	
	/* can be called using an array */
	yourFunction([ 42, 'Hi!', true ])
	
	/* can be called using an object */
	yourFunction({
		yourInt: 42, 
		yourString: 'Hi!',
		yourValue: true
	})
```

*Each of these results in `Object { yourInt: 42, yourString: 'Hi!', yourValue: true }`*

***

#### Optional values can be left out:

```
	yourFunction(42, true)
```

*This results in `Object { yourInt: 42, yourValue: true }`*

***

#### Required values can be left out if `defaultValue` is present:

```
	yourFunction(true) 
```

*This results in `Object { yourInt: 0, yourValue: true }`*

***

#### Parsing Arguments, Objects or Arrays is handled the same way:

```js
	/* Each of the following has the same result */
	let input = arguments // can accepts arguments
	let input = [ 1972, 'Portland' ] // can accept an array
	let input = { year: 1972, city: 'Portland' } // can accept an object

	let args = Galactic.args({
		city: 'string',
		year: 'integer'
	}).parse(input)
```

*This results in `Object {year: 1972, city: 'Portland'}`*

***

#### And you can reuse the validator:

```js
	let validator = Galactic.args(schema, onInvalid)
	let method = validator.wrap(handler)
	let args1 = validator.parse(arguments)
	let args2 = validator.parse(object)
	let args3 = validator.parse(array)
```


### Detailed Usage

***

#### What is a schema?

*A `Galactic.args(...)` schema is simply an object that defines how data should look.*

*Here's an example of a basic schema:*

```
{ /* this is a schema */
	foo: { /* this is a pattern inside the schema */
		type: 'string',
		defaultValue: 'Bunnies!'
	}
}
```

*This particular schema states the argument `foo` must be a String, if its not, or its `undefined`, then its value will be "Bunnies!".*

***

#### Schema: Pattern arguments

*Schemas are made up of one or more patterns. The following describes the functionality that patterns provide. Also the following is ordered based on how data flows through a pattern.*

1. **`type`:** String | constructor ***[optional]***

  * If `type` is a String then it's assumed to be a [shortcut](#schema-shorthand-patterns).
  
  * If `type` is an Object constructor then an equality check is performed.
	
	*PRO TIP—See [Galactic.is(...)](https://github.com/mudcube/Galactic.js/blob/master/galactic.is.md) for a more in depth guide to the type checking engine used in `Galactic.args(...)`, this comes along with information on how to create your own custom type validators.*

2. **`validate`:** RegExp | Function ***[optional]***

  * If `type` is matched, or `type` is not set, then a `validate` function can be used to more specifically verify the match. This `validate` function receives 2-arguments `(value, type)` and is expected to return true or false.

3. **`defaultValue`:** ***[optional]***

  * If our value has failed the `validate` then the `defaultValue` is used. This can be any type of variable.

4. **`optional`:** Boolean ***[optional]***

  * If optional = true then failing the `type` match and the `validate` test won't emit an error.
  
5. **'catch':** Function ***[optional]***

  * If there is no match or no `defaultValue` then `catch` is called recieving the object `({ key, value, type })`. Effectively, `catch` allows you to do 2-things:
  
    1. Throw a more specific error 
    2. Prevent an error by returning anything but `undefined`, essentially functioning as a contextual 'defaultValue''.

6. **`transform`:** Function ***[optional]***

  * If a match is found then `transform` is applied. This function receives 2-arguments `(value, type)`. The return value can be any type of variable.

At this point we're all validated and nicely formatted!

***

#### Schema: Shorthand patterns

```js
	let validator = Galactic.args({
		bat: Number, // `bat` must be a Number
		bear: 'number', // `bear` must also be a Number
		beer: 'number?', // `beer` is an optional Number
		beard: 'number=42', // `beard` is an optional Number with defaultValue of 42
		badger: 'number|boolean' // `badger` must be a Number or a Boolean
	})
```

***

#### Schema: Longhand patterns are more flexible

```js
	let validator = Galactic.args({
		egg: {
			type: 'string',
			optional: true, // this is an optional argument
			validate: /\s{7,42}/, // this is a RegExp matching a string between 7-42 chars
			transform: function (value, type) { // this formats the results of valid matches to uppercase
				return value.toUpperCase()
			}
		},
		caterpillar: {
			type: 'blob|arraybuffer',
			validate: function (value, type) {
				if (type === 'blob') {
					return value.size > 42
				} else { // type === 'arraybuffer'
					return value.byteLength > 42
				}
			},
			catch: function (e) { // this throws custom error message
				throw `${e.key} expected a Blob or ArrayBuffer but received a ${e.type}`;
			}
		},
		butterfly: {
			type: 'string',
			catch: function (e) { // this catches errors or returns a contextual defaultValue
				if (e.type === 'string') { // string is not long enough
					return `${e.value}paddingSoWeAreLongEnough`
				} else { // argument is not of type String
					return 'aWholeNewString'
				}
			}
		}
	})
```

***

#### Multiple schema patterns can be used in combination:

```js
	let validator = Galactic.args([
		{ // schema pattern #1
			cake: 'number'
		},
		{ // schema pattern #2 overrides any keys in pattern #1
			cake: 'boolean'
		}
	])
```

***

#### Nested validation on Objects looks like this:

```js
	let validator = Galactic.args({
		'food': 'object',
		'food.muffin': 'object',
		'food.muffin.delightful': 'boolean'
	})
```

***

#### Nested validation on Arrays looks like this:

```js
	let validator = Galactic.args({
		'foods': 'array',
		'foods.$.name': 'string' // loops through 'foods' and ensures the value of the 'name' key is a string
	})
```

### Prior art

***

* [args.js](https://joebain.github.io/args.js)
	* *Fairly comparable features to Galactic.args(...) with a different interface.*
* [simpleschema](https://github.com/aldeed/meteor-simple-schema)
	* *Powerful schema validation for Meteor apps. Galactic.args(...) nested validation was patterned after it, thanks!*
* [validate.js](https://github.com/rickharrison/validate.js)
	* *Form validation library in VanillaJS.*
