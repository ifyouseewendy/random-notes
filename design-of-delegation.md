# Behavior Delegation

Source: [You Don't Know JS: this & Object Prototypes - Chapter 6: Behavior Delegation](https://github.com/getify/You-Dont-Know-JS/blob/master/this %26 object prototypes/ch6.md)

## 1. My understanding

Considering we don't actually have `class` in Javascript, but we want the benefit of behaviour sharing around code entities. Javascript employs behaviour delegation as the `[[Prototype]]`mechanism. It kinda differs from the traditional class-instance  thinking, but it's still in the spectrum of OO, as a form of plain objects linking \(delegation\) instead of inheritance.

Classical inheritance is a code arrangement technique. For the cost of arranging objects in a hierarchy, you get message delegation for free. Delegation arranges objects in a horizontal space \(side-by-side as peers\) instead of a vertical hierarchy. So, can I say one outweighs another between behaviour delegation and traditional class theory? No, they are different assumptions that we don't have true `class` in Javascript.

Behavior delegation looks like a side-effect outcome on the way Javascript strives to simulate class-oriented code to meet the expectations of most OO developers. For instance, `new` creates an automatic message delegation just like inheritance, name of `constructor` , introducing `class`in ES6. It's probable that people added `prototype` aiming to simulate class behaviours.

Anyway, behaviour delegation works and I consider it as the right mental model to illustrate the chaos in Javascript, which is much better than the contrived class thinking.

## 2. Background

JavaScript is **almost unique **among languages as perhaps the only language with the right to use the label "object oriented", because it's one of a very short list of languages where **an object can be created directly, without a class at all.**

In JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. **JavaScript just has objects**. In JavaScript, we don't make _copies_ from one object \("class"\) to another \("instance"\). **We make links between objects.**

```js
function Foo() {
    // ...
}

var a = new Foo();
var b = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
Object.getPrototypeOf( b ) === Foo.prototype; // true
```

When`a`is created by calling`new Foo()`, one of the things \(see Chapter 2 for all four steps\) that happens is that`a`gets an internal`[[Prototype]]`link to the object that`Foo.prototype`is pointing at. **We end up with two objects, linked to each other.**

The actual mechanism, the essence of what's important to the functionality we can leverage in JavaScript, is **all about objects being linked to other objects.**

### Compared to traditional inheritance

In class-oriented languages, multiple copies \(aka, "instances"\) of a class can be made, like stamping something out from a mold. But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]]`link to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quit_e _**linked**.

"inheritance" \(and "prototypal inheritance"\) and all the other OO terms just do not make sense when considering how JavaScript _actually_ works \(not just applied to our forced mental models\).

Instead, "delegation" is a more appropriate term, because **these relationships are not **_**copies**_** but delegation **_**links**_.

### Prototypal Inheritance && Differential Inheritance

This mechanism is often called "**prototypal inheritance**" \(we'll explore the code in detail shortly\), which is commonly said to be the dynamic-language version of "classical inheritance". The word "inheritance" has a very strong meaning \(see Chapter 4\), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the _actually nearly opposite_ behavior in JavaScript has left in its wake nearly two decades of miry confusion."Inheritance" implies a _copy_ operation, and JavaScript doesn't copy object properties \(natively, by default\). Instead, JS creates a link between two objects, where one object can essentially _delegate_ property/function access to another object. "**Delegation**" is a much more accurate term for JavaScript's object-linking mechanism.

Another term which is sometimes thrown around in JavaScript is "**differential inheritance**". The idea here is that we describe an object's behavior in terms of what is _different_ from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle \(engine, etc\).

But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B`is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" \(gaps in, or lack of, definition\) that delegation _can_ take over and, on the fly, "fill them in" with delegated behavior.

## 3. Create delegations by `Object.create`

`Object.create(..)` creates a "new" object out of thin air, and links that new object's internal `[[Prototype]]`to the object you specify.

```js
var a = { name: "wendi" };
var b = Object.create(a);

b.name // "wendi"
b.__proto__ === a // true
```

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

## 4. Towards Delegation-Oriented Design

Pseudo-code for class theory

```js
class Task {
    id;

    // constructor `Task()`
    Task(ID) { id = ID; }
    outputTask() { output( id ); }
}

class XYZ inherits Task {
    label;

    // constructor `XYZ()`
    XYZ(ID,Label) { super( ID ); label = Label; }
    outputTask() { super(); output( label ); }
}

class ABC inherits Task {
    // ...
}
```

Pseudo-code for delegation theory

```js
var Task = {
    setID: function(ID) { this.id = ID; },
    outputID: function() { console.log( this.id ); }
};

// make `XYZ` delegate to `Task`
var XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
    this.setID( ID );
    this.label = Label;
};

XYZ.outputTaskDetails = function() {
    this.outputID();
    console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

### Avoid shadowing \(naming things the same\) if at all possible

With the class design pattern, we intentionally named`outputTask`the same on both parent \(`Task`\) and child \(`XYZ`\), so that we could take advantage of overriding \(polymorphism\). In behavior delegation, we do the opposite: **we avoid if at all possible naming things the same **at different levels of the`[[Prototype]]`chain \(called **shadowing**\), because having those name collisions creates awkward/brittle syntax to disambiguate references, and we want to avoid that if we can.

This design pattern calls for less of general method names which are prone to overriding and instead more of descriptive method names, specific to the type of behavior each object is doing.**This can actually create easier to understand/maintain code**, because the names of methods \(not only at definition location but strewn throughout other code\) are more obvious \(self documenting\).

Setting properties on an object was more nuanced than just adding a new property to the object or changing an existing property's value. Usually, shadowing is more complicated and nuanced than it's worth, **so you should try to avoid it if possible**.

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

That's the reason why we use delegation on prototype chain, we should avoid using the same name as traditional class inheritance would do.

### Save state on delegators

In general, with`[[Prototype]]`delegation involved, **you want state to be on the delegators**\(`XYZ`,`ABC`\), not on the delegate \(`Task`\). We benefit it from the implicit call-site `this`binding rules.

### Comparison

OO style

```js
function Foo(who) {
    this.me = who;
}
Foo.prototype.identify = function() {
    return "I am " + this.me;
};

function Bar(who) {
    Foo.call( this, who );
}
Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

OO style features `constructor` which introduces a lot of extra details that you don't \_technically \_need to know at all times.

![](/assets/OO.png)

OLOO style

```js
var Foo = {
    init: function(who) {
        this.me = who;
    },
    identify: function() {
        return "I am " + this.me;
    }
};

var Bar = Object.create( Foo );

Bar.speak = function() {
    alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
```

OLOO-style code has \_vastly less stuff \_to worry about, because it embraces the **fact **that the only thing we ever really cared about was the **objects linked to other objects**.

![](/assets/OLOO.png)

