# "Class"

### Misconception

There's a peculiar kind of behavior in JavaScript that has been shamelessly abused for years to _hack _something that _looks _like "classes". JS developers have strived to simulate as much as they can of class-orientation.

### Design

In JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. **JavaScript just has objects**. In JavaScript, we don't make _copies _from one object \("class"\) to another \("instance"\). **We make **_**links **_**between objects.**

In fact, JavaScript is **almost unique **among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where **an object can be created directly, without a class at all.**

```js
function Foo() {
	// ...
}

var a = new Foo();
var b = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
Object.getPrototypeOf( b ) === Foo.prototype; // true
```

When`a`is created by calling`new Foo()`, one of the things \(see Chapter 2 for all_four_steps\) that happens is that`a`gets an internal`[[Prototype]]`link to the object that`Foo.prototype`is pointing at. **We end up with two objects, linked to each other.**





Compared to traditional inheritance

In class-oriented languages, multiple copies \(aka, "instances"\) of a class can be made, like stamping something out from a mold. But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]] `_link _to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite _**linked**_. 

#### 

#### Prototypal Inheritance && Differential Inheritance

This mechanism is often called "**prototypal inheritance**" \(we'll explore the code in detail shortly\), which is commonly said to be the dynamic-language version of "classical inheritance". The word "inheritance" has a very strong meaning \(see Chapter 4\), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the _actually nearly opposite _behavior in JavaScript has left in its wake nearly two decades of miry confusion."Inheritance" implies a _copy _operation, and JavaScript doesn't copy object properties \(natively, by default\). Instead, JS creates a link between two objects, where one object can essentially _delegate _property/function access to another object. "**Delegation**" is a much more accurate term for JavaScript's object-linking mechanism.

Another term which is sometimes thrown around in JavaScript is "**differential inheritance**". The idea here is that we describe an object's behavior in terms of what is _different _from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle \(engine, etc\).  
But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B `is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" \(gaps in, or lack of, definition\) that delegation _can _take over and, on the fly, "fill them in" with delegated behavior.

#### 

## `new` and `constructor`

In fact, the secret, which eludes most JS developers, is that the`new Foo()`function calling had really almost nothing _direct _to do with the process of creating the link. **It was sort of an accidental side-effect.**`new Foo()`is an indirect, round-about way to end up with what we want:**a new object linked to another object**. \(Can we get what we want in a more _direct _way? **Yes! **The hero is`Object.create(..)`.\)





Object.create \( create delegation \)



## 

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

### Do we have to capitalize the constructor function? NO!

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

### What is exactly a "constructor"?

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the**`new`**keyword **in front of it.

Functions aren't constructors, but function calls are "constructor calls" if and only if`new`is used.

### Does "constructor" mean "was constructed by"? NO!

The fact is,`.constructor`on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls`.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

### Is `.constructor `reliable to be used as a reference? NO!

Some arbitrary object-property reference like`a1.constructor`cannot actually be _trusted_ to be the assumed default function reference. Moreover, as we'll see shortly, just by simple omission,`a1.constructor`can even end up pointing somewhere quite surprising and insensible.`a1.constructor`is extremely unreliable, and an unsafe reference to rely upon in your code.**Generally, such references should be avoided where possible.**

`.constructor`is not a magic immutable property. It _is_ non-enumerable \(see snippet above\), but its value is writable \(can be changed\), and moreover, you can add or overwrite \(intentionally or accidentally\) a property of the name`constructor`on any object in any`[[Prototype]]`chain, with any value you see fit.

```js
function Foo() {}
var a = new Foo();
a.constructor  === Foo // true

Foo.prototype.constructor = () => { console.log('a') }
a.constructor === Foo // false
```

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

That's a lot of manual work to fix`.constructor`. Moreover, all we're really doing is perpetuating the misconception that "constructor" means "was constructed by". That's an _expensive_ illusion.

