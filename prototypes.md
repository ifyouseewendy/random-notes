# Prototypes

### What is a prototype?

Objects in JavaScript have an internal property, denoted in the specification as`[[Prototype]]`, which is simply a reference to another object. Almost all objects are given a non-`null`value for this property, at the time of their creation.

### How to get an object's prototype?

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

### What is the `prototype` ?

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

So, `Arary`, `Function`, `Object`are all functions. I should admit that this refreshes my impression on JS. I know functions are first-class citizen in JS but it seems that it is built on functions.

### What's the difference between `__proto__` and `prototype`?

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

### What's process of method lookup via prototype chain?

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

## "Class"

Design

Misconception

new and constructor



Object.create \( create delegation \)

Try to avoid shadowing if possible

Setting properties on an object was more nuanced than just adding a new property to the object or changing an existing property's value.

Usually, shadowing is more complicated and nuanced than it's worth, **so you should try to avoid it if possible**.

```js
var anotherObject = {
    a: 2
};

var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "a" ); // false

myObject.a++; // oops, implicit shadowing!

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty( "a" ); // true
```

Though it may appear that`myObject.a++`should \(via delegation\) look-up and just increment the`anotherObject.a`property itself _in place _, instead the`++`operation corresponds to`myObject.a = myObject.a + 1`.

## "Class"

In fact, JavaScript is **almost unique **among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where an object can be created directly, without a class at all.

### "Class" Functions

All functions by default get a public, non-enumerable \(see Chapter 3\) property on them called`prototype`, which points at an otherwise arbitrary object.

```js
function Foo() {
    // ...
}

Foo.prototype; // { }
```

Each object created from calling`new Foo()`\(see Chapter 2\) will end up \(somewhat arbitrarily\)`[[Prototype]]`-linked to this "Foo dot prototype" object.

```js
function Foo() {
    // ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
a.__proto__ === Foo.prototype // true

a.__proto__ // Object {constructor: function}
```

In fact, the secret, which eludes most JS developers, is that the`new Foo()`function calling had really almost nothing \_direct \_to do with the process of creating the link. **It was sort of an accidental side-effect.**`new Foo()`is an indirect, round-about way to end up with what we want: **a new object linked to another object**. Can we get what we want in a more \_direct \_way? **Yes! **The hero is`Object.create(..)`.

### "Constructors"

```js
function Foo() {
    // ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

The`Foo.prototype`object by default \(at declaration time on line 1 of the snippet!\) gets a public, non-enumerable \(see Chapter 3\) property called`.constructor`, and this property is a reference back to the function \(`Foo`in this case\) that the object is associated with.

By convention in the JavaScript world, "class"es are named with a capital letter, so the fact that it's`Foo`instead of`foo`is a strong clue that we intend it to be a "class". But the capital letter doesn't mean _**anything **_**at all **to the JS engine.

```js
function foo() {}

new foo() // foo {}

foo.prototype // Object {constructor: function}
```

In reality,`Foo`is no more a "constructor" than any other function in your program. Functions themselves are **not **constructors. However, when you put the`new`keyword in front of a normal function call, that makes that function call a "constructor call". In fact,`new`sort of hijacks any normal function and calls it in a fashion that constructs an object, **in addition to whatever else it was going to do**.

```js
function NothingSpecial() {
    console.log( "Don't mind me!" );
}

var a = new NothingSpecial();
// "Don't mind me!"

a; // {}
```

`NothingSpecial`is just a plain old normal function, but when called with`new`, it_constructs\_an object, almost as a side-effect, which we happen to assign to_`a`_. The **call **was a \_constructor call_, but`NothingSpecial`is not, in and of itself, a_constructor_.

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the**`new`**keyword **in front of it.

Functions aren't constructors, but function calls are "constructor calls" if and only if`new`is used.

If you create a new object, and replace a function's default`.prototype`object reference, the new object will not by default magically get a`.constructor`on it.

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

What's happening?`a1`has no`.constructor`property, so it delegates up the`[[Prototype]]`chain to`Foo.prototype`. But that object doesn't have a`.constructor`either \(like the default`Foo.prototype`object would have had!\), so it keeps delegating, this time up to`Object.prototype`, the top of the delegation chain.\_That \_object indeed has a`.constructor`on it, which points to the built-in`Object(..)`function.

Of course, you can add`.constructor`back to the`Foo.prototype`object, but this takes manual work, especially if you want to match native behavior and have it be non-enumerable.

```js
function Foo() { /* .. */ }

Foo.prototype = { /* .. */ }; // create a new prototype object

// Need to properly "fix" the missing `.constructor`
// property on the new object serving as `Foo.prototype`.
// See Chapter 3 for `defineProperty(..)`.
Object.defineProperty( Foo.prototype, "constructor" , {
    enumerable: false,
    writable: true,
    configurable: true,
    value: Foo    // point `.constructor` at `Foo`
} );
```

That's a lot of manual work to fix`.constructor`. Moreover, all we're really doing is perpetuating the misconception that "constructor" means "was constructed by". That's an \_expensive \_illusion.

Illusion: "constructor" means "was constructed by"

The fact is,`.constructor`on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls`.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

Illusion: `.constructor`is reliable to be used as a reference

```js
function Foo() {}
var a = new Foo();
a.constructor  === Foo // true

Foo.prototype.constructor = () => { console.log('a') }
a.constructor === Foo // false
```

`.constructor`is not a magic immutable property. It \_is \_non-enumerable \(see snippet above\), but its value is writable \(can be changed\), and moreover, you can add or overwrite \(intentionally or accidentally\) a property of the name`constructor`on any object in any`[[Prototype]]`chain, with any value you see fit.

Some arbitrary object-property reference like`a1.constructor`cannot actually be \_trusted \_to be the assumed default function reference. Moreover, as we'll see shortly, just by simple omission,`a1.constructor`can even end up pointing somewhere quite surprising and insensible.`a1.constructor`is extremely unreliable, and an unsafe reference to rely upon in your code.**Generally, such references should be avoided where possible.**

`Object.create(..)`\_creates \_a "new" object out of thin air, and links that new object's internal`[[Prototype]]`to the object you specify \(`Foo.prototype`in this case\).

In other words, that line says: "make a_new_'Bar dot prototype' object that's linked to 'Foo dot prototype'."

---

In class-oriented languages, multiple **copies**\(aka, "instances"\) of a class can be made, like stamping something out from a mold. As we saw in Chapter 4, this happens because the process of instantiating \(or inheriting from\) a class means, "copy the behavior plan from that class into a physical object", and this is done again for each new instance.

But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that`[[Prototype]]`_link\_to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite _**linked**\_.

In JavaScript, we don't make \_copies \_from one object \("class"\) to another \("instance"\). We make \_links \_between objects.

JS developers have strived to simulate as much as they can of class-orientation.

**Misconception**

prototypal inheritance

The word "inheritance" has a very strong meaning \(see Chapter 4\), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the \_actually nearly opposite \_behavior in JavaScript has left in its wake nearly two decades of miry confusion. Because of the confusion and conflation of terms, I believe the label "prototypal inheritance" itself \(and trying to mis-apply all its associated class-orientation terminology, like "class", "constructor", "instance", "polymorphism", etc\) has done **more harm than good **in explaining how JavaScript's mechanism \_really \_works.

"Inheritance" implies a \_copy \_operation, and JavaScript doesn't copy object properties \(natively, by default\). Instead, JS creates a link between two objects, where one object can essentially \_delegate \_property/function access to another object. "Delegation" \(see Chapter 6\) is a much more accurate term for JavaScript's object-linking mechanism.

differential inheritance

The idea here is that we describe an object's behavior in terms of what is \_different \_from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle \(engine, etc\).

If you try to think of any given object in JS as the sum total of all behavior that is_available\_via delegation, and**in your mind you flatten**all that behavior into one tangible \_thing_, then you can \(sorta\) see how "differential inheritance" might fit.

But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object`B`is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" \(gaps in, or lack of, definition\) that delegation \_can \_take over and, on the fly, "fill them in" with delegated behavior.

