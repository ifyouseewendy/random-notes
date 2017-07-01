# Types in JS

Primary types \(language types\)

* number
* boolean
* string
* null
* undefined
* object

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



