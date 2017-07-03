### Design

In JavaScript, there are no abstract patterns/blueprints for objects called "classes" as there are in class-oriented languages. **JavaScript just has objects**. In JavaScript, we don't make _copies \_from one object \("class"\) to another \("instance"\). **We make **_**links **\_**between objects.**

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

In class-oriented languages, multiple copies \(aka, "instances"\) of a class can be made, like stamping something out from a mold. But in JavaScript, there are no such copy-actions performed. You don't create multiple instances of a class. You can create multiple objects that `[[Prototype]]`_link \_to a common object. But by default, no copying occurs, and thus these objects don't end up totally separate and disconnected from each other, but rather, quite _**linked**\_.

#### 

#### Prototypal Inheritance && Differential Inheritance

This mechanism is often called "**prototypal inheritance**" \(we'll explore the code in detail shortly\), which is commonly said to be the dynamic-language version of "classical inheritance". The word "inheritance" has a very strong meaning \(see Chapter 4\), with plenty of mental precedent. Merely adding "prototypal" in front to distinguish the \_actually nearly opposite \_behavior in JavaScript has left in its wake nearly two decades of miry confusion."Inheritance" implies a \_copy \_operation, and JavaScript doesn't copy object properties \(natively, by default\). Instead, JS creates a link between two objects, where one object can essentially \_delegate \_property/function access to another object. "**Delegation**" is a much more accurate term for JavaScript's object-linking mechanism.

Another term which is sometimes thrown around in JavaScript is "**differential inheritance**". The idea here is that we describe an object's behavior in terms of what is \_different \_from a more general descriptor. For example, you explain that a car is a kind of vehicle, but one that has exactly 4 wheels, rather than re-describing all the specifics of what makes up a general vehicle \(engine, etc\).  
But just like with "prototypal inheritance", "differential inheritance" pretends that your mental model is more important than what is physically happening in the language. It overlooks the fact that object `B`is not actually differentially constructed, but is instead built with specific characteristics defined, alongside "holes" where nothing is defined. It is in these "holes" \(gaps in, or lack of, definition\) that delegation \_can \_take over and, on the fly, "fill them in" with delegated behavior.

#### 

### Object.create \( create delegation \)



