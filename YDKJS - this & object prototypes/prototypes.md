# Prototypes

Source: [You Don't Know JS: this & Object Prototypes - Chapter 5: Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/master/this %26 object prototypes/ch5.md)

## 1. My understanding

After reading this part, I realize that `Arary`, `Function`, `Object`are all functions. I should admit that this refreshes my impression on JS. I know functions are first-class citizen in JS but it seems that it is all built on functions. Every object is created by functions:

```js
// simple primitives are auto boxing: new Number(1)
var number = 1                       

// object created by constructor
var date = new Date("2017-07-01");

// object literal
var obj = {} // is equivalent to: Object.create(Object.prototype);

var obj = { foo: "hello" } // is equivalent to

Object.create(
  Object.prototype,
  {
    foo: {
      writable: true,
      configurable: true,
      value: 'hello'
    }
  }
)
```

## 2. What is a prototype?

Objects in JavaScript have an internal property, denoted in the specification as`[[Prototype]]`, which is simply a reference to another object. Almost all objects are given a non-`null`value for this property, at the time of their creation.

## 3. How to get an object's prototype?

via `__proto__`or `Object.getPrototypeOf`

```js
var a = { name: "wendi" };
a.__proto__ === Object.prototype // true
Object.getPrototypeOf(a) === Object.prototype // true

function Foo() {};
var b = new Foo();
b.__proto__ === Foo.prototype
b.__proto__.__proto__ === Object.prototype
```

So where is `__proto__`defined? `Object.prototype.__proto__`

We could roughly envision `__proto__` implemented like this

```js
Object.defineProperty( Object.prototype, "__proto__", {
    get: function() {
        return Object.getPrototypeOf( this );
    },
    set: function(o) {
        // setPrototypeOf(..) as of ES6
        Object.setPrototypeOf( this, o );
        return o;
    }
} );
```

**Note: **The JavaScript community unofficially coined a term for the double-underscore, specifically the leading one in properties like`__proto__`: "dunder". So, the "cool kids" in JavaScript would generally pronounce`__proto__`as "dunder proto".

## 4. What is the `prototype` ?

`prototype` is an object automatically created as a special property of a **function**, which is used to establish the delegation \(inheritance\) chain, aka prototype chain.

When we create a function `a`, `prototype` is automatically created as a special property on `a` and saves the function code on as the `constructor` on `prototype`.

```js
function Foo() {};
Foo.prototype // Object {constructor: function}
Foo.prototype.constructor === Foo // true
```

I'd love to consider this property as the place to store the properties \(including methods\) of a function object. That's also the reason why utility functions in JS are defined like `Array.prototype.forEach()` , `Function.prototype.bind()`, `Object.prototype.toString().`

Why to emphasize the property of a **function**?

```js
{}.prototype // undefined;
(function(){}).prototype // Object {constructor: function}

// The example above shows object does not have the prototype property.
// But we have Object.prototype, which implies an interesting fact that
typeof Object === "function"
var obj = new Object();
```

## 5. What's the difference between `__proto__` and `prototype`?

`__proto__`a reference works on every **object** to refer to its `[[Prototype]]`property.

`prototype` is an object automatically created as a special property of a **function**, which is used to store the properties \(including methods\) of a function object.

With these two, we could mentally map out the prototype chain. Like this picture illustrates:

![](/assets/__proto__-vs-prototype.png)

```js
function Foo() {}
var b = new Foo();

b.__proto__ === Foo.prototype // true
Foo.__proto__ === Function.prototype // true
Function.prototype.__proto__ === Object.prototype // true
```

Refer to: [\_\_proto\_\_ VS. prototype in JavaScript](https://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)

## 6. What's process of method lookup via prototype chain?

```js
Object.foo = function() { console.log("foo"); };

function Foo() {}
Foo.prototype.bar = function() { this.foo(); console.log("bar"); };

var a = new Foo();
a.baz = function(){ this.bar(); console.log("baz"); };
a.baz();

// foo
// bar
// baz
```

The top-end of every _normal _`[[Prototype]]`chain is the built-in `Object.prototype`. This object includes a variety of common utilities used all over JS.

```js
Object.prototype

constructor: function Object()
hasOwnProperty: function hasOwnProperty()
isPrototypeOf: function isPrototypeOf()
propertyIsEnumerable: function propertyIsEnumerable()
toLocaleString: function toLocaleString()
toString: function toString()
valueOf: function valueOf()
__defineGetter__: function __defineGetter__()
__defineSetter__: function __defineSetter__()
__lookupGetter__: function __lookupGetter__()
__lookupSetter__: function __lookupSetter__()
get __proto__: function __proto__()
set __proto__: function __proto__()
```



