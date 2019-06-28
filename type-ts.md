This blog post is a quick introduction to TypeScript’s notation for static types.

What you’ll learn  [#](#what-youll-learn)
-----------------------------------------

After reading this post, you should be able to understand what the following code means:

    interface Array<T> {
      concat(...items: Array<T[] | T>): T[];
      reduce<U>(
        callback: (state: U, element: T, index: number, array: T[]) => U,
        firstState?: U): U;
      ···
    }
    

If you think this is cryptic – then I agree with you. But (as I hope to prove) this notation is relatively easy to learn. And once you understand it, it gives you immediate, precise and comprehensive summaries of how code behaves. No need to read long descriptions in English.

Trying out the code examples  [#](#trying-out-the-code-examples)
----------------------------------------------------------------

TypeScript has [an online playground](http://www.typescriptlang.org/play/). In order to get the most comprehensive checks, you should switch on everything in the “Options” menu. This is equivalent to running the TypeScript compiler in `--strict` mode.

Specifying the comprehensiveness of type checking  [#](#specifying-the-comprehensiveness-of-type-checking)
----------------------------------------------------------------------------------------------------------

I always use TypeScript with the most comprehensive setting, `--strict`. Without it, programs are slightly easier to write, but you also lose many benefits of static type checking. Currently, this setting enables the following sub-settings:

*   `--noImplicitAny`: If TypeScript can’t infer a type, you must specify it. This mainly applies to parameters of functions and methods: With this settings, you must annotate them.
*   `--noImplicitThis`: Complain if the type of `this` isn’t clear.
*   `--alwaysStrict`: Use JavaScript’s strict mode whenever possible.
*   `--strictNullChecks`: `null` is not part of any type (other than its own type, `null`) and must be explicitly mentioned if it is a acceptable value.
*   `--strictFunctionTypes`: stronger checks for function types.
*   `--strictPropertyInitialization`: If a property can’t have the value `undefined`, then it must be initialized in the constructor.

More info: chapter “[Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)” in the TypeScript Handbook.

Types  [#](#types)
------------------

In this blog post, a type is simply a set of values. The JavaScript language (not TypeScript!) has 7 types:

*   Undefined: the set with the only element `undefined`.
*   Null: the set with the only element `null`.
*   Boolean: the set with the two elements `false` and `true`.
*   Number: the set of all numbers.
*   String: the set of all strings.
*   Symbol: the set of all symbols.
*   Object: the set of all objects (which includes functions and arrays).

All of these types are _dynamic_: you can use them at runtime.

TypeScript brings an additional layer to JavaScript: _static types_. These only exist when compiling or type-checking source code. Each storage location (variable or property) has a static type that predicts its dynamic values. Type checking ensures that these predictions come true. And there is a lot you can check _statically_ (without running the code). If, for example the parameter `x` of a function `f(x)` has the static type `number`, then the function call `f('abc')` is illegal, because the parameter `'abc'` has the wrong static type.

Type annotations  [#](#type-annotations)
----------------------------------------

A colon after a variable name starts a _type annotation_: the _type signature_ after the colon describes what values the variable can have. For example, the following line tells TypeScript that `x` will only ever store numbers:

    let x: number;
    

You may wonder if `x` being initialized with `undefined` doesn’t violate the static type. TypeScript gets around this problem by not letting you read `x` before you assign a value to it.

Type inference  [#](#type-inference)
------------------------------------

Even though every storage location has a static type in TypeScript, you don’t always have to explicitly specify it. TypeScript can often infer it. For example, if you write:

    let x = 123;
    

Then TypeScript infers that `x` has the static type `number`.

Describing types  [#](#describing-types)
----------------------------------------

What comes after the colon of a type annotation is a so-called type expression. These range from simple to complex and are created as follows.

Basic types are valid type expressions:

*   Static types for JavaScript’s dynamic types:
    *   `undefined`, `null`
    *   `boolean`, `number`, `string`
    *   `symbol`
    *   `object`.
    *   Note: value `undefined` vs. type `undefined` (depends on locations)
*   TypeScript-specific types:
    *   `Array` (not technically a type in JS)
    *   `any` (the type of all values)
    *   Etc.

Note that “`undefined` as a value” and “`undefined` as a type” are both written as `undefined`. Depending on where you use it, it is interpreted as a value or as a type. The same is true for `null`.

You can create more type expressions by combining basic types via _type operators_, which combine types similarly to how the operators _union_ (`∪`) and intersection (`∩`) combine sets.

The following sections explain a few of the type operators that TypeScript offers.

Array types  [#](#array-types)
------------------------------

Arrays are used in the following two roles in JavaScript (and sometimes a mix of the two):

*   Lists: All elements have the same type. The length of the Array varies.
*   Tuple: The length of the Array is fixed. The elements do not necessarily have the same type.

### Arrays as lists  [#](#arrays-as-lists)

There are two ways to express the fact that the Array `arr` is used as a list whose elements are all numbers:

    let arr: number[] = [];
    let arr: Array<number> = [];
    

Normally, TypeScript can infer the type of a variable if there is an assignment. In this case, you actually have to help it, because with an empty Array, it can’t determine the type of the elements.

We’ll get back to the angle brackets notation (`Array<number>`) later.

### Arrays as tuples  [#](#arrays-as-tuples)

If you store a two-dimensional point in an Array then you are using that Array as a tuple. That looks as follows:

    let point: [number, number] = [7, 5];
    

In this case, you don’t need the type annotation.

Another example for tuples is the result of `Object.entries(obj)`: an Array with one \[key, value\] pair for each property of `obj`.

    > Object.entries({a:1, b:2})
    [ [ 'a', 1 ], [ 'b', 2 ] ]
    

The type of the result of `Object.entries()` is:

    Array<[string, any]>
    

Function types  [#](#function-types)
------------------------------------

This is an example of a function type:

    (num: number) => string
    

This type comprises all functions that accept a single parameter, a number, and return a string. Let’s use this type in a type annotation (`String` is used as a function here):

    const func: (num: number) => string = String;
    

Again, we normally wouldn’t use a type annotation here, because TypeScript knows the type of `String` and can therefore infer the type of `func`.

The following code is a more realistic example:

    function stringify123(callback: (num: number) => string) {
      return callback(123);
    }
    

We are using a function type to describe the parameter `callback` of `stringify123()`. Due to this type annotation, TypeScript rejects the following function call.

    f(Number);
    

But it accepts the following function call:

    f(String);
    

### Result types of function declarations  [#](#result-types-of-function-declarations)

It’s a good practice to annotate all parameters of functions. You can also specify the result type (but TypeScript is quite good at inferring it):

    function stringify123(callback: (num: number) => string): string {
      const num = 123;
      return callback(num);
    }
    

#### The special result type `void`  [#](#the-special-result-type-void)

`void` is a special result type for functions: It tells TypeScript that the function always returns `undefined` (explicitly or implicitly):

    function f1(): void { return undefined } // OK
    function f2(): void { } // OK
    function f3(): void { return 'abc' } // error
    

### Optional parameters  [#](#optional-parameters)

A question mark after an identifier means that the parameter is optional. For example:

    function stringify123(callback?: (num: number) => string) {
      const num = 123;
      if (callback) {
        return callback(num); // (A)
      }
      return String(num);
    }
    

If you run TypeScript in `--strict` mode, it will only let you make the function call in line A if you check beforehand that `callback` hasn’t been omitted.

#### Parameter default values  [#](#parameter-default-values)

TypeScript supports [ES6 parameter default values](http://exploringjs.com/es6/ch_parameter-handling.html#sec_parameter-default-values):

    function createPoint(x=0, y=0) {
      return [x, y];
    }
    

Default values make parameters optional. You can usually omit type annotations, because TypeScript can infer the types. For example, it can infer that `x` and `y` both have the type `number`.

If you wanted to add type annotations, that would look as follows.

    function createPoint(x:number = 0, y:number = 0) {
      return [x, y];
    }
    

### Rest types  [#](#rest-types)

You can also use [the ES6 rest operator](http://exploringjs.com/es6/ch_parameter-handling.html#sec_rest-parameters) for TypeScript parameter definitions. The type of the corresponding parameter must be an Array:

    function joinNumbers(...nums: number[]): string {
        return nums.join('-');
    }
    joinNumbers(1, 2, 3); // '1-2-3'
    

Union types  [#](#union-types)
------------------------------

In JavaScript, variables occasionally have one of several types. To describe those variables, you use _union types_. For example, in the following code, `x` is either of type `null` or of type `number`:

    let x = null;
    x = 123;
    

The type of `x` can be described as `null|number`:

    let x: null|number = null;
    x = 123;
    

The result of the type expression `s|t` is the set-theoretic union of the types `s` and `t` (which, as we have seen earlier, as both sets).

Let’s rewrite function `stringify123()`: This time, we don’t want the parameter `callback` to be optional. It should always be mentioned. If callers don’t want to provide a function, they have to explicitly pass `null`. That is implemented as follows.

    function stringify123(
      callback: null | ((num: number) => string)) {
      const num = 123;
      if (callback) { // (A)
        return callback(123); // (B)
      }
      return String(num);
    }
    

Note that, once again, we have to check if `callback` is actually a function (line A), before we can make the function call in line B. Without the check, TypeScript would report an error.

### Optional vs. `undefined|T`  [#](#optional-vs-undefinedt)

Optional parameters of type `T` and parameters of type `undefined|T` are quite similar. (As an aside, the same is true for optional properties.)

The main difference is that you can omit optional parameters:

    function f1(x?: number) { }
    f1(); // OK
    f1(undefined); // OK
    f1(123); // OK
    

But you can’t omit parameters of type `undefined|T`:

    function f2(x: undefined | number) { }
    f2(); // error
    f2(undefined); // OK
    f2(123); // OK
    

### The values `null` and `undefined` are not generally included in types  [#](#the-values-null-and-undefined-are-not-generally-included-in-types)

In many programming languages, `null` is part of all types. For example, whenever the type of a parameter is `String` in Java, you can pass `null` and Java won’t complain.

In contrast, in TypeScript, `undefined` and `null` are handled by separate, disjoint types. You need a type union such as `undefined|string` and `null|string`, if you want to allow them.

Typing objects  [#](#typing-objects)
------------------------------------

Similarly to Arrays, objects play two roles in JavaScript (that are occasionally mixed and/or more dynamic):

*   Records: A fixed amount of properties that are known at development time. Each property can have a different type.
    
*   Dictionaries: An arbitrary amount of properties whose names are not known at development time. All property keys (strings and/or symbols) have the same type, as do the property values.
    

We’ll ignore objects-as-dictionaries in this blog post. As an aside, Maps are usually a better choice for dictionaries, anyway.

### Typing objects-as-records via interfaces  [#](#typing-objects-as-records-via-interfaces)

Interfaces describe objects-as-records. For example:

    interface Point {
      x: number;
      y: number;
    }
    

One big advantage of TypeScript’s type system is that it works _structurally_, not _nominally_. That is, interface `Point` matches all objects that have the appropriate structure:

    function pointToString(p: Point) {
      return `(${p.x}, ${p.y})`;
    }
    pointToString({x: 5, y: 7}); // '(5, 7)'
    

In contrast, Java’s nominal type system requires classes to _implement_ interfaces.

### Optional properties  [#](#optional-properties)

If a property can be omitted, you put a question mark after its name:

    interface Person {
      name: string;
      company?: string;
    }
    

### Methods  [#](#methods)

Interfaces can also contain methods:

    interface Point {
      x: number;
      y: number;
      distance(other: Point): number;
    }
    

Type variables and generic types  [#](#type-variables-and-generic-types)
------------------------------------------------------------------------

With static typing, you have two levels:

*   Values exist at the _object level_.
*   Types exist at a _meta level_.

Similarly:

*   Normal variables exist at the object level.
*   There are also _type variables_, which exist at the meta level. They are variables whose values are types.

Normal variables are introduced via `const`, `let`, etc. Type variables are introduced via angle brackets (`< >`). For example, the following code contains the type variable `T`, as introduced via `<T>`.

    interface Stack<T> {
      push(x: T): void;
      pop(): T;
    }
    

You can see that the type parameter `T` appears twice inside the body of `Stack`. Therefore, this interface can intuitively understood as follows:

*   `Stack` is a stack of values that all have a given type `T`. You must fill in `T` whenever you mention `Stack`. We’ll see how, next.
*   Method `.push()` accepts values of type `T`.
*   Method `.pop()` returns values of type `T`.

If you use `Stack`, you must assign a type to `T`. The following code shows a dummy stack, whose only purpose is to match the interface.

    const dummyStack: Stack<number> = {
      push(x: number) {},
      pop() { return 123 },
    };
    

### Example: Maps  [#](#example-maps)

Maps are typed generically in TypeScript. For example:

    const myMap: Map<boolean,string> = new Map([
      [false, 'no'],
      [true, 'yes'],
    ]);
    

### Type variables for functions  [#](#type-variables-for-functions)

Functions (and methods) can introduce type variables, too:

    function id<T>(x: T): T {
      return x;
    }
    

You use this function as follows.

    id<number>(123);
    

Due to type inference, you can also omit the type parameter:

    id(123);
    

### Passing on type parameters  [#](#passing-on-type-parameters)

Functions can pass on their type parameters to interfaces, classes, etc.:

    function fillArray<T>(len: number, elem: T) {
      return new Array<T>(len).fill(elem);
    }
    

The type variable `T` appears three times in this code:

*   `fillArray<T>`: introduce the type variable
*   `elem: T`: use the type variable, pick it up from the argument.
*   `Array<T>`: pass on `T` to the `Array` constructor.

That means: we don’t have to explicitly specify the type `T` of `Array<T>` – it is inferred from parameter `elem`:

    const arr = fillArray(3, '*');
      // Inferred type: string[]
    

Conclusion  [#](#conclusion)
----------------------------

Let’s use what we have learned to understand the piece of code we have seen earlier:

    interface Array<T> {
      concat(...items: Array<T[] | T>): T[];
      reduce<U>(
        callback: (state: U, element: T, index: number, array: T[]) => U,
        firstState?: U): U;
      ···
    }
    

This is an interface for an Array whose elements are of type `T` that we have to fill in whenever we use this interface:

*   method `.concat()` has zero or more parameters (defined via the rest operator). Each of those parameters has the type `T[]|T`. That is, it is either an Array of `T` values or a single `T` value.
    
*   method `.reduce()` introduces its own type variable, `U`. `U` expresses the fact that the following entities all have the same type (which you don’t need to specify, it is inferred automatically):
    
    *   Parameter `state` of `callback()` (which is a function)
    *   Result of `callback()`
    *   Optional parameter `firstState` of `.reduce()`
    *   Result of `.reduce()`
    
    `callback` also gets a parameter `element` whose type has the same type `T` as the Array elements, a parameter `index` that is a number and a parameter `array` with `T` values.
    

Further reading  [#](#further-reading)
--------------------------------------

*   Book (free to read online): “[Exploring ES6](http://exploringjs.com/es6/)”
*   “[ECMAScript Language Types](https://tc39.github.io/ecma262/#sec-ecmascript-language-types)” in the ECMAScript specification.
*   “[TypeScript Handbook](http://www.typescriptlang.org/docs/handbook/basic-types.html)”: is well-written and explains various other kinds of types and type operators that TypeScript supports.
*   The TypeScript repository has [type definitions for the complete ECMAScript standard library](https://github.com/Microsoft/TypeScript/blob/master/lib/). Reading them is an easy way of practicing the type notation.
