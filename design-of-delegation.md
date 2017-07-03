## Design

TBD

In JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. **JavaScript just has objects**. In JavaScript, we don't make _copies \_from one object \("class"\) to another \("instance"\). **We make **_**links between objects**

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

When`a`is created by calling`new Foo()`, one of the things \(see Chapter 2 for all\_four\_steps\) that happens is that`a`gets an internal`[[Prototype]]`link to the object that`Foo.prototype`is pointing at. **We end up with two objects, linked to each other.**

### Compared to traditional inheritance

In class-oriented languages, multiple copies \(aka, "instances"\) of a class can be made, like stamping something out from a mold. But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]]`_link to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite _**linked**\_.

"inheritance" \(and "prototypal inheritance"\) and all the other OO terms just do not make sense when considering how JavaScript _actually _works \(not just applied to our forced mental models\).

Instead, "delegation" is a more appropriate term, because these relationships are not _copies _but delegation **links**.

#### Prototypal Inheritance && Differential Inheritance

This mechanism is often called "**prototypal inheritance**" \(we'll explore the code in detail shortly\), which is commonly said to be the dynamic-language version of "classical inheritance". The word "inheritance" has a very strong meaning \(see Chapter 4\), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the \_actually nearly opposite \_behavior in JavaScript has left in its wake nearly two decades of miry confusion."Inheritance" implies a \_copy \_operation, and JavaScript doesn't copy object properties \(natively, by default\). Instead, JS creates a link between two objects, where one object can essentially \_delegate \_property/function access to another object. "**Delegation**" is a much more accurate term for JavaScript's object-linking mechanism.

Another term which is sometimes thrown around in JavaScript is "**differential inheritance**". The idea here is that we describe an object's behavior in terms of what is _different_ from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle \(engine, etc\).  
But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B`is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" \(gaps in, or lack of, definition\) that delegation \_can \_take over and, on the fly, "fill them in" with delegated behavior.

## Delegations by `Object.create`

`Object.create(..)`\_creates \_a "new" object out of thin air, and links that new object's internal `[[Prototype]]`to the object you specify.

### How to make plain object delegations?

```js
var foo = {
    something: function() {
        console.log( "Tell me something good..." );
    }
};

var bar = Object.create( foo );

bar.__proto__ === foo // true
bar.something(); // Tell me something good...
```

### How to make delegations to perform "prototypal inheritance"?

```js
function Foo() {};
Foo.prototype.foo = function() { console.log("foo"); };
function Bar() {};

Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.__proto__ === Foo.prototype // true

var b = new Bar();
b.foo(); // foo
```

Inspection: `Bar.prototype` has changed

```js
function Foo() {};
Foo.prototype.foo = function() { console.log("foo"); };
function Bar() {};

Bar.prototype // Object {constructor: function}
Bar.prototype = Object.create(Foo.prototype);
Bar.prototype // Foo {}
Bar.prototype.bar = function() { console.log("bar"); };
Bar.prototype // Foo {bar: function}
```

Inspection: `Bar.prototype` is not a reference \(separated\) to `Foo.prototype`

```js
function Foo() {};
Foo.prototype.foo = function() { console.log("foo"); };
function Bar() {};

Bar.prototype = Object.create(Foo.prototype);
Bar.prototype.bar = function() { console.log("bar"); };

Foo.prototype // Object {foo: function, constructor: function}
Bar.prototype // Function {bar: function}
```

Why not?

```js
Bar = Object.create(Foo.prototype)
// Cause it ends up Bar is no longer a function object.

Bar.prototype = Object.create(Foo)
// This links Bar.prototype to Foo, which is the function object. Foo.foo() is not a function.

Bar.prototype = Foo.prototype;
// It just makes Bar.prototype be another reference to Foo.prototype

Bar.prototype = new Foo();
// It creates a new object, but Foo might have unexpected behaviours in constructor calling
```

ES6-standardized techniques

```js
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

### How to envision your own `Object.create`?

This polyfill shows a very basic idea without handling the second parameter `propertiesObject`.

```js
function createAndLinkObject(o) {
    function F(){}
    F.prototype = o;
    return new F();
}
```

Polyfill on [Object.create - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

```js
if (typeof Object.create != 'function') {
  Object.create = (function(undefined) {
    var Temp = function() {};
    return function (prototype, propertiesObject) {
      if(prototype !== Object(prototype)) {
        throw TypeError(
          'Argument must be an object, or null'
        );
      }
      Temp.prototype = prototype || {};
      var result = new Temp();
      Temp.prototype = null;
      if (propertiesObject !== undefined) {
        Object.defineProperties(result, propertiesObject); 
      } 

      // to imitate the case of Object.create(null)
      if(prototype === null) {
         result.__proto__ = null;
      } 
      return result;
    };
  })();
}
```



