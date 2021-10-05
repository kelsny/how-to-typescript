# How to type magic

> People always ask, "How do you do your type magic stuff?", so I'm going to try and answer them.

[Ew skip the boring sh\*t](#part-1)

#### Prerequisites

In this book I'm going to assume you know the following:

- [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) - Fundamentals, ES6 features, and the standard library.
- [TypeScript](http://www.typescriptlang.org/) - What it is, why use it, basic syntax.
- [Node.js](https://nodejs.org/en/) - What it is, `global` and `process`, native modules' API.

If you aren't quite caught up on these topics, do so now so that the book does not become confusing or overwhelming for you.
I'd hate for someone to leave more confused than they started.

### Foreword

Ok, first I'd like to admit not everyone says "How do you do your type magic stuff?".
Sometimes they give me a weird quizzical look and say, "What the f\*ck is the use of this?".
Well the use is a great challenge for your logic and method for practicing actually useful TypeScript skills.
Useful? Where'd that come from? 
It's useful because if you can do type magic, you can apply your type magic to real-world scenarios!
You'll see later throughout this book when we apply a new idea, concept, or pattern to something that might be used in production.

Second, I'm definitely not that good at type magic.
To be quite honest, most of the time I just grab whatever sh\*t I can think of to solve a problem and throw it in to see if it does something remotely similar to what I want.
I'd like to think I'm pretty good but then I go to [Type Challenges](https://github.com/type-challenges/type-challenges/issues) and...

It's too painful to talk about.

### Table of contents

# Part 1 - Walk before you run

It's important to walk before you run, so let's review TypeScript's unique type system first.
If you already think you got it well enough to get started, [skip ahead](#part-2)!
In Part 1 we will skim over the fundamentals of TypeScript types, and in the next part, we'll finally create some type magic!

## Chapter 1 - Crawling

And here we have our humble beginnings of TypeScript.

```ts
let foo = 123;

foo = "456";
```

Some very smart people at Microsoft got together and said, "Wow this code sucks!". A little later, TypeScript was born (don't quote me on this).
Ok, that probably wasn't how the language was created, but the point still stands; this code is complete garbage!
**There is almost no reason for a variable to have its type changed!** If a variable's type changes, other parts of the code will either forget it changes type or have to be severely modified to account for its dynamic type.

#### Primitives

You're probably familiar with the 6 primitives in JavaScript: `string`, `number`, `boolean`, `bigint`, `symbol`, and `undefined` (yeah f\*ck `null`).
In TypeScript there are 4 more: `any`, `unknown`, `never`, `void`.

Let's go through each of them.

**`any`**

The `any` type encompasses all possible types in TypeScript. Anything can be assigned to `any` and `any` can be assigned to anything.
Of course, you should never use `any` in any circumstance, but as we'll see later, you might need it in type magic for some nefarious purposes.

**`unknown`**

You could think of `unknown` as a type-safe version of `any`. This type is used when the type of something is unknown at runtime.
A common example would be user input or a request body in an API route. You have to narrow down the type before you use it, unlike `any`!
`unknown` is used many times in type magic, because it has some useful properties we'll explore later.

**`never`**

The `never` type is almost like the polar opposite of `any`.
`never` cannot be assigned to anything (except for `never` itself), and everything can't be assigned to `never` (except for `never` itself yet again).
It's often used when there is no type to use, or can be used. An example would be a function that always throws an error:

```ts
function iNeverReturn() {
  throw new Error(`This function does nothing.`);
}
```

The return type of `iNeverReturn` is inferred as `never`, since it will never return anything. Another common use of `never` is in `process.exit`:

```ts
function iNeverReturn() {
  process.exit(1);
}
```

Since the process will be terminated, the function will never return anything.

As we will see later, `never` also is extremely useful in type magic.

**`void`**

Finally, we have the `void` type. This type is inferred when the function does not explicitly return (or is supposed to return nothing, like `console.log`). Like `never`, it's selectively assignable. There's not much to this type, in my opinion, so we'll just gloss over it for now.

#### Boxed Primitives

JavaScript also includes boxed primitives. Boxed primitives are object representations of primitives. Wrappers, if you will.
When you call a method on a primitive like a string, the string is temporarily wrapped in an object, then the object's method is called.
After that, the object is discarded and forgotten.

```js
const primitive = "hello world";         // string

const boxed = new String("hello world"); // String

const constructor = String;              // StringConstructor
```

Keep in mind that there is a *really* big difference between a primitive, boxed primitive, and the boxed primitive constructor's types.
A primitive is lowercase (`string`), boxed primitive is capitalized (`String`), and the constructor has `Constructor` appended to it (`StringConstructor`).

There is a small difference with user-defined classes, however, and we'll get to that soon.

Like `any`, there is almost no reason to use the boxed primitive (constructor) type either, because using both `string` and `String` will lead to some pretty confusing and hard-to-find errors.

#### Object types

What's the use of a JavaScript type system if you can't define types for objects?

The syntax for defining object types is very similar to vanilla JavaScript:

```ts
type MyType = {
  foo: string;
  bar: number;
  baz: {
    qux: boolean;
  }
}
```

There is also the `object` type, which is also pretty taboo like `any`, because most things can be assigned to it.
Another one like `object` is `{}`. And also like primitives, there is `Object` and `ObjectConstructor`.

You *might* yse `ObjectConstructor` because of `Object`'s built-in methods, for example, `Object.entries` (in types, `ObjectConstructor["entries"]`).

Most of the times, however, you'll always use the standard syntax for defining object types.

#### Operators

A type system without any operators to aid us in developing said types? Unheard of!
Thankfully, TypeScript includes several operators to ease the use of type composition.

They're all pretty simple and intuitive, so I'll go through all of them extremely fast.

**Union (`|`)**

No, not bitwise OR! It's the union operator! A union is basically an OR operator. This thing can be this type, OR this type, OR this type, etc.
Unions allow more flexibility (and some design patterns) for the type system to flourish.

```ts
type StringOrNumber = string | number;
```

**Intersection (`&`)**

Again, it's not bitwise AND, it's the intersection operator. This operator takes two types and merges them, if it can (if it can't it'll produce `never`).
Intersections allow the type system to have complex type compositions.

**Type of something (`typeof`)***

Ok, this one's like the JavaScript `typeof` operator, but it retrieves the inferred or explicitly defined type of a value.

```ts
import Module from "library";

type TypeOfImportedThing = typeof Module;
```

**Is subtype (`extends`)**

<>

# Part 2 - The new horizon

```
// The new way to look at types
```

# Part 3 - Design & Develop

```
// Common patterns amongst the type magic community
```

# Part 4 - Extensions

```
// Real-world scenarios and resources for further reading
```

### Afterword

### Contributing
