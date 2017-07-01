# Objects in JS

## 1. Type

Primary types \(language types\)

* number
* boolean
* string
* null
* undefined
* object

Many people mistakenly claim "everything in JavaScript is an object", but this is incorrect. Objects are one of the 6 \(or 7, depending on your perspective\) primitive types. Objects have sub-types, including`function`, and also can be behavior-specialized, like`[object Array]`as the internal label representing the array object sub-type.

| value | Primitives | Object Sub-types | typeof value |
| :--- | :--- | :--- | :--- |
| 1 | number | Number | "number" |
| true | boolean | Boolean | "boolean" |
| "hello" | string | String | "string" |
| null | null | - | "object" \(!!\) |
| undefined | undefined | - | "undefined" |
| { name: "wendi" } | object | Object | "object" |
| \[1, 2, 3\] | object | Array | "object" |
| function\(\) {} | object | Function | "function" \(!!\) |
| /^abc/ | object | RegExp | "object" |
| new Date\("2017-07-01"\) | object | Date | "object" |
| new Error\("shit"\) | object | Error | "object" |

What are object sub-types?

In JS, object sub-types are actually just built-in functions. Each of these built-in functions can be used as a constructor \(that is, a function call with the new operator\), with the result being a newly constructed object of the sub-type.

Why do we need object sub-types?

The primitive value `"I am a string"`is not an object, it's a primitive literal and immutable value. To perform operations on it, such as checking its length, accessing its individual character contents, etc, a`String`object is required.the language automatically coerces a`"string"`primitive to a`String`object when necessary, which means you almost never need to explicitly create the Object form.

How to remember?

Excluding from the self-defined object, we can always use `typeof` first to check out the primary types and then use `instanceof` to find out its object sub-types.

## 2. Contents

Objects are collections of key/value pairs. The values can be accessed as properties, via`.propName`or`["propName"]`syntax. Whenever a property is accessed, the engine actually invokes the internal default`[[Get]]`operation \(and`[[Put]]`for setting values\), which not only looks for the property directly on the object, but which will traverse the`[[Prototype]]`chain \(see Chapter 5\) if not found. 

Properties have certain characteristics that can be controlled through property descriptors, such as`writable`and`configurable`. In addition, objects can have their mutability \(and that of their properties\) controlled to various levels of immutability using`Object.preventExtensions(..)`,`Object.seal(..)`, and`Object.freeze(..)`. 

Properties don't have to contain values -- they can be "accessor properties" as well, with getters/setters. They can also be either_enumerable_or not, which controls if they show up in`for..in`loop iterations, for instance.

### Properties

In objects, property names are **always **strings. If you use any other value besides a `string`\(primitive\) as the property, it will first be converted to a string.

```js
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"];				// "foo"
myObject["3"];					// "bar"
myObject["[object Object]"];                  // "baz"
```

### Computed Property Names

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

### Arrays

Arrays are objects. **Be careful**: If you try to add a property to an array, but the property name looks like a number, it will end up instead as a numeric index \(thus modifying the array contents\):

```js
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";

myArray.length;	// 4

myArray[3];		// "baz"
```

### Duplicating Objects

```js
function anotherFunction() { /*..*/ }

var anotherObject = {
	c: true
};

var anotherArray = [];

var myObject = {
	a: 2,
	b: anotherObject,	// reference, not a copy!
	c: anotherArray,	// another reference!
	d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

It's hard to tell which of shallow and deep copy is right without the use case.

One subset solution is that objects which are JSON-safe \(that is, can be serialized to a JSON string and then re-parsed to an object with the same structure and values\) can easily be _duplicated _with:

```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

A shallow copy is fairly understandable and has far less issues, so ES6 has now defined `Object.assign(..)` for this task. `Object.assign(..)` takes a target object as its first parameter, and one or more source objects as its subsequent parameters. It iterates over all the _enumerable_ \(see below\), _owned keys \(immediately present\)_ on the source object\(s\) and copies them \(via = assignment only\) to target.

```js
var newObj = Object.assign( {}, myObject );

newObj.a;						// 2
newObj.b === anotherObject;		// true
newObj.c === anotherArray;		// true
newObj.d === anotherFunction;	// true
```

### Property Descriptors

Prior to ES5, the JavaScript language gave no direct way for your code to inspect or draw any distinction between the characteristics of properties, such as whether the property was read-only or not. But as of ES5, all properties are described in terms of a **property descriptor**.

```js
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
//    value: 2,
//    writable: true,
//    enumerable: true,
//    configurable: true
// }
```

We can use`Object.defineProperty(..)`to add a new property, or modify an existing one \(if it's`configurable`!\), with the desired characteristics.

```js
var myObject = {};

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumerable: true
} );

myObject.a; // 2
```

I consider this as something about plumbing facts, which features some higher level operations. Like

**Seal**: `Object.seal(..)` creates a "sealed" object, which means it takes an existing object and essentially calls `Object.preventExtensions(..)` on it, but also marks all its existing properties as `configurable:false`.

**Freeze**: `Object.freeze(..)` creates a frozen object, which means it takes an existing object and essentially calls `Object.seal(..)` on it, but it also marks all "data accessor" properties as `writable:false`, so that their values cannot be changed.

For details, check [this section](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch3.md#property-descriptors)

### \[ \[ Get \] \]

```js
var myObject = {
	a: 2
};

myObject.a; // 2
```

The`myObject.a`is a property access, but it doesn't _just _look in `myObject `for a property of the name `a`, as it might seem. According to the spec, the code above actually performs a`[[Get]]`operation \(kinda like a function call:`[[Get]]()`\) on the`myObject`. The default built-in`[[Get]]`operation for an object _first _inspects the object for a property of the requested name, and if it finds it, it will return the value accordingly. 

One important result of this`[[Get]]`operation is that if it cannot through any means come up with a value for the requested property, it instead returns the value`undefined`\(instead of a`ReferenceError`\).

Define an **accessor descriptor **\(getter and putter\)

```js
var myObject = {
	// define a getter for `a`
	get a() {
		return this._a_;
	},

	// define a setter for `a`
	set a(val) {
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a; // 4
```

### Existence

We showed earlier that a property access like`myObject.a`may result in an`undefined`value if either the explicit`undefined`is stored there or the`a`property doesn't exist at all. So, if the value is the same in both cases, how else do we distinguish them?

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

The`in`operator will check to see if the property is _in _the object, or if it exists at any higher level of the`[[Prototype]]`chain object traversal \(see Chapter 5\). By contrast,`hasOwnProperty(..)`checks to see if _only _`myObject`has the property or not, and will _not _consult the`[[Prototype]]`chain.

`hasOwnProperty(..)`is accessible for all normal objects via delegation to`Object.prototype`\(see Chapter 5\). But it's possible to create an object that does not link to`Object.prototype`\(via`Object.create(null)`-- see Chapter 5\). In this case, a method call like`myObject.hasOwnProperty(..)`would fail.

In that scenario, a more robust way of performing such a check is`Object.prototype.hasOwnProperty.call(myObject,"a")`, which borrows the base`hasOwnProperty(..)`method and uses_explicit`this`binding_\(see Chapter 2\) to apply it against our`myObject`.

## 3. Iteration

The`for..in`loop iterates over the list of enumerable properties on an object \(including its`[[Prototype]]`chain\). But what if you instead want to iterate over the values?

`for..in`loops applied to arrays can give somewhat unexpected results, in that the enumeration of an array will include not only all the numeric indices, but also any enumerable properties. It's a good idea to use`for..in`loops _only _on objects, and traditional`for`loops with numeric index iteration for the values stored in arrays.

ES5 also added several iteration helpers for arrays, including `forEach(..)`, `every(..)`, and `some(..)`.

* `forEach(..)`will iterate over all values in the array, and ignores any callback return values.
* `every(..)`keeps going until the end _or _the callback returns a`false`\(or "falsy"\) value, whereas
* `some(..)`keeps going until the end _or _the callback returns a `true`\(or "truthy"\) value.

As contrasted with iterating over an array's indices in a numerically ordered way \(`for`loop or other iterators\), the order of iteration over an object's properties is **not guaranteed **and may vary between different JS engines. **Do not rely **on any observed ordering for anything that requires consistency among environments, as any observed agreement is unreliable.

You can also iterate over **the values **in data structures \(arrays, objects, etc\) using the ES6`for..of`syntax, which looks for either a built-in or custom`@@iterator`object consisting of a`next()`method to advance through the data values one at a time.

