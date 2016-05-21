# Galactic.args(...)

#### The easy way to handle arguments! No dependancies. Vanilla JS.
 
 *... or "The guide to Nonviolent Communications for Javascript Arguments Handling" ;)*

### Basic Usage

***
#### Galactic.args(...) wraps a Function to ensure input data follows schema:

```js
	let validator = Galactic.args(schema, onInvalid)
```

***

#### The validator contains two functions:

* .parse(...)
	* **.parse(object)** *— If parse(...) is called, an object is returned with data consistency based on you schema.*

* .wrap(...)
	* **.wrap(handler)** *— If wrap(...) is called, a function is returned. This is a wrapper around your function that ensures data consistency, while maintaining the 'handler's `this` scope.*

***

#### Here's an example of wrapping a function:

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

#### Now you can call yourFunction(...) in multiple ways with the same results!

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

#### Required values can be left out when a 'defaultValue' is present:

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

#### Schema

*A Galactic.args(...) schema is simply an object that defines how data should look.*

*Here's an example of a basic schema:*

```
{ /* this is a schema */
	foo: { /* this is a pattern inside the schema */
		type: 'string',
		defaultValue: 'Bunnies!'
	}
}
```

*This particular schema states the argument `foo` must be a string, if its not, or its `undefined`, then its value will be "Bunnies!".*

***

#### Schema: Pattern arguments

*Schemas are made up of one or more patterns. The following describes the functionality that patterns provide. Also the following is ordered based on how data flows through a pattern.*

1. **'type':** String | constructor ***[optional]***

  *If 'type' is a string then it's assumed to be a [shortcut](#schema-shorthand-patterns).*
  
  *If 'type' is an object constructor then an equality check is performed.*
	
	*PRO TIP—See [Galactic.is(...)](https://github.com/mudcube/Galactic.js/blob/master/galactic.is.md) for a more in depth guide to the type checking engine used in Galactic.args(...), along with information on how to create your own custom type validators.*

2. **'validate':** RegExp | Function ***[optional]***

  *If 'type' is matched, or is not set, then a 'validate' function can be used to more specifically verify the match. This function receives 2-arguments (value, type). It's expected to return true or false.*

3. **'defaultValue':** ***[optional]***

  *If our value has failed 'validate' then the 'defaultValue' is used. This can be any type of variable.*

4. **'optional':** Boolean ***[optional]***

  *If optional = true then failing the 'type' match and the 'validate' test won't emit an error.*
  
5. **'catch':** Function ***[optional]***

  *If there is no match or no 'defaultValue' then 'catch' is called. This function receives an object ({ key, value, type }). Catch allows you to do two things: (1) Throw a more specific error (2) Prevent an error by returning anything but undefined, essentially functioning as a contextual 'defaultValue''.*

6. **'transform':** Function ***[optional]***

  *If a match is found then 'transform' is applied. This function receives 2-arguments (value, type). The return value can be any type of variable. At this point we're all validated and nicely formatted!*

***

#### Schema: Shorthand patterns

*Here's a few examples to highlight the features of using shortcuts:*

```js
	let validator = Galactic.args({
		bat: Number, // required number
		bear: 'number', // required number
		beer: 'number?', // optional number
		beard: 'number=42', // optional number with defautlValue of 42
		badger: 'number|boolean' // required number or boolean
	})
```

***

#### Schema: Longhand patterns are more flexible

```js
	let validator = Galactic.args({
		egg: {
			type: 'string',
			optional: true, // optional argument
			validate: /\s{7,42}/, // RegExp matching a string between 7-42 chars
			transform: function (value, type) { // formats the results of valid matches to uppercase
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
			catch: function (e) { // throws custom error message
				throw `${e.key} expected a Blob or ArrayBuffer but received a ${e.type}`;
			}
		},
		butterfly: {
			type: 'string',
			catch: function (e) { // catch error and return contextual defaultValue
				if (e.type === 'string') { // string is not long enough
					return `${e.value}paddingSoWeAreLongEnough`
				} else { // is not of type 'string'
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
