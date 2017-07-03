# "Class"

## Misconception

There's a peculiar kind of behavior in JavaScript that has been shamelessly abused for years to \_hack \_something that \_looks \_like "classes". JS developers have strived to simulate as much as they can of class-orientation.

## "Constructors"

```js
function Foo() {
    // ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

The`Foo.prototype`object by default \(at declaration time on line 1 of the snippet!\) gets a public, non-enumerable property called`.constructor`, and this property is a reference back to the function \(`Foo`in this case\) that the object is associated with.

### Does "constructor" mean "was constructed by"? NO!

The fact is,`.constructor`on an object arbitrarily points, by default, at a function who, reciprocally, has a reference back to the object -- a reference which it calls`.prototype`. The words "constructor" and "prototype" only have a loose default meaning that might or might not hold true later. The best thing to do is remind yourself, "constructor does not mean constructed by".

### What is exactly a "constructor"?

In other words, in JavaScript, it's most appropriate to say that a "constructor" is **any function called with the**`new`**keyword **in front of it. Functions aren't constructors, but function calls are "constructor calls" if and only if`new`is used.

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

`NothingSpecial`is just a plain old normal function, but when called with`new`, it_constructs\_an object, almost as a side-effect, which we happen to assign to_`a`_. The **call **was a constructor call_, but`NothingSpecial`is not, in and of itself, a_constructor_.

### Is `.constructor`reliable to be used as a reference? NO!

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

### What happened when we call`new` ?

```js
function New(func) {
    var res = {};
    if (func.prototype !== null) {
        res.__proto__ = func.prototype;
    }
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return res;
}

var obj = New(A, 1, 2);
// equals to
var obj = new A(1, 2);
```

1. It creates a new object. The type of this object, is simply _object_
2. It sets this new object's internal, inaccessible, _\[\[prototype\]\] _\(i.e. **\_\_proto\_\_ **\) property to be the constructor function's external, accessible, _prototype _object \(every function object automatically has a _prototype _property\).
3. It makes the `this `variable point to the newly created object.
4. It executes the constructor function, using the newly created object whenever `this `is mentioned.
5. It returns the newly created object, unless the constructor function returns a non-`null `object reference. In this case, that object reference is returned instead.

Reference

* [What is the 'new' keyword in JavaScript?](https://stackoverflow.com/questions/1646698/what-is-the-new-keyword-in-javascript)
* [new operator - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)

## Introspection

### `instanceof`

```js
a instanceof Foo; // true
```

The`instanceof`operator takes a plain object as its left-hand operand and a**function**as its right-hand operand. The question`instanceof`answers is:**in the entire`[[Prototype]]`chain of`a`, does the object arbitrarily pointed to by`Foo.prototype`ever appear?**

What if you have two arbitrary objects, say`a`and`b`, and want to find out if _the objects _are related to each other through a`[[Prototype]]`chain?

```js
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
	function F(){}
	F.prototype = o2;
	return o1 instanceof F;
}

var a = {};
var b = Object.create( a );

isRelatedTo( b, a ); // true
```

### `Object.prototype.isPrototypeOf()`

```js
Foo.prototype.isPrototypeOf( a ); // true
```

The question`isPrototypeOf(..)`answers is:**in the entire`[[Prototype]]`chain of`a`, does`Foo.prototype`ever appear?**

What if you have two arbitrary objects, say`a`and`b`, and want to find out if _the objects _are related to each other through a`[[Prototype]]`chain?

```js
// Simply: does `a` appear anywhere in
// `b`s [[Prototype]] chain?
a.isPrototypeOf( b );
```



