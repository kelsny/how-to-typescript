**Translations:** [中文](./translations/zh)

**Styles:** [Informal](./styles/informal)

# How to TypeScript

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

#### Literal types

Small thing to note here: `"any-string-possible"` is a valid type. `0` is a valid type. 
In fact, all primitive literals are a valid type (except for a few, like `Infinity` and `NaN`).
So when you see me using literals, I'm probably using them as a type literal.

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

```ts
type Foo = { foo: string };
type Bar = { bar: string };

type FooAndBar = Foo & Bar;                    // { foo: string; bar: string }

type ImpossibleIntersection = string & number; // is never
```

**Type of something (`typeof`)***

Ok, this one's like the JavaScript `typeof` operator, but it retrieves the inferred or explicitly defined type of a value.

```ts
import Module from "library";

type TypeOfImportedThing = typeof Module;
```

**Is subtype (`extends`)**

This one isn't the `extends` like you use in a class, although it does relate to it in a way. No, `extends` can be used in one of 2 ways:
- As a generic type constraint (you will see what I mean)
- As a way to check if a type extends another type (if a type is a subtype of another type)

This one takes a little more work to understand, so I'll leave it up to you if you want to learn all about it right now.

Ok, that was the fundamentals, let's talk about a little more advanced types next.

## Chapter 2 - Walking

That was a nice review of the basics, now let's use them to create more complex types.

#### Functions

Wait... we haven't covered functions yet! How are we supposed to annotate and manipulate function types? Ay don't worry TypeScript got you covered!
Let's first take a look at the basic syntax for annotating a function.

```ts
function foo(bar: string, baz: number): boolean {
  return Number(bar) === baz;
}
```

This simple function checks if a string is equal to a number numerically (side tangent: [Equality operators: Why I always use strict equal/not-equal](./tangents/equality.md)). and its parameters and return type are both annotated. Note that TypeScript could've inferred the return type here, and that explicitly annotating it is optional.

The type of this function expressed in a function expression type would be `(bar: string, baz: number) => boolean`. Pretty straight-forward.

What about a method on an object? TypeScript would show you the type (in a tooltip) as a function expression type. For defining an object type with a method, you've got two ways.

```ts
type MyObj = {
  foo(param1: string, param2: string): string;
  bar: () => number;
};
```

There's two different ways, because we've got two different functions (plain and arrow). Note that `() => ...` does not represent an arrow function.
Both ways are simply expressing a method on an object.

Since JavaScript is such a hot mess of a language, you can change the `this` value of a function (`bind`, `call`, `apply`)!
But how do you model this with TypeScript? 
TypeScript allows you to annotate the `this` type using `this` as the first parameter in a function:

```ts
type Binded = (this: { foo: string }) => void;
```

Now when a function's type is marked as `Binded`, TypeScript will pretend `this` is `{ foo: string }`.

Another small thing, when using literal function types with operators, you must wrap them in parentheses:

```ts
type UnionOfFns = () => void | () => never;     // ! ERROR
type UnionOfFns = (() => void) | (() => never); // ^ Good!
```

#### Arrays and tuples

Ah wait. Even if arrays are just special objects, they deserve their own way to define them as types...

You might be familiar with array types in other languages. For example, in Java, to create an array of length 10 with only integers:

```java
int[10] myArray;
```

Or maybe even just

```java
int[] myArray;
```

to declare an array.

Multi-dimensonal arrays follow from this, of course: `int[][]`.

Some languages also support tuples, namely Python. TypeScript supports ways to represent both of these important structures;

Array types are declared in the same fashion as Java.

```ts
type AliasForStringArray = string[]
```

Multi-dimensional

```ts
type Array2D = number[][]
```

Now for something that might blow you statically typed compiled language people's minds.

It's important to think about `[]` as an *operator*. It takes its operand on the left and wraps it as an array type. In this way, we can compose complex array types:

```ts
type AReallyComplexArrayType = { key: ((string | number)[] & { depth: number })[] }[]
```

Note the use of parentheses; `(string | number)[]` is definitely not the same as `string | number[]`.

Tuples are declared in a different fashion. They look like array literals in code, but instead of values, its types in their place.

```ts
type Position = [number, number];
```

You can also name each element:

```ts
type Position = [x: number, y: number];
```

Which is extremely useful to abuse when using dark type magic.

In the next section we'll explore modifiers for arrays and tuples.

#### Modifiers

Alright so we've got a way to represent almost every type possible in JavaScript, now it's about flexibility and more accurate modeling of some JavaScript objects.
We'll first start of with some extra operators, then finish it off with some more advanced types.

**`keyof`**

The `keyof` operator simply takes its only operand, and outputs a union of its keys. For example, `keyof { foo: string }` gives me `"foo"`, while `keyof { foo: string; bar: string }` gives me `"foo" | "bar"`.
Not much by itself, but it can be used with another cool thing TypeScript allows us to do, which we'll see very soon.

**`readonly`**

Yep. We've got `readonly`. You can make any object readonly just with this special keyword. It can be an object's property, the object itself, arrays, etc.

```ts
type ReadonlyFoo = { readonly foo: string; bar: number };

type ReadonlyArrayOfNumbers = readonly number[];
```

You can do some experimentation to see what's up and how it works (hint: there is a difference between making a value readonly and a property readonly).

**`?`**

?. What's this? I should give more context.
Prefix `:` in an object after a key with `?` to make it optional.
In older versions of TypeScript, this was equivalent to adding `| undefined` to the type of the key.

```ts
type OptionalProp = {
  foo?: string;
};
```

**`-`**

Oh another weird symbol? What's *this* one for?
I'll answer that. It's for removing `?` and `readonly` in an object. We'll see how this can be used in the next one.

**`in`**

Alas, we have `in`, which allows us to create mapped types.
Personally, I think mapped types are a little badly named.
But basically, they're called mapped types because the type is computed by mapping a key to a type.

```ts
type MappedNumbers = {
  [K in "foo" | "bar"]: number;
}; // { foo: number; bar: number }
```

We can also use `keyof` here:

```ts
type FooBar = { foo: string; bar: string };

type MappedNumbers = {
  [K in keyof FooBar]: number;
};
```

Along with `-` to remove optionality of properties, as well as readonly-ness (?):

```ts
type FooBar = { readonly foo: string; readonly bar: string };

type RequiredFooBar = {
  -readonly [K in keyof FooBar]-?: string;
};
```

Mapped types come in handy a few times, make sure to practice them yourself!

## Chapter 3 - Running

Ahh. We've been waiting for this one, haven't we? The infamous generics, notoriously hard to first understand, to grasp the concept and uses of this feature.

I'll first give you the most common example: trying to generalize a class to work with many types (probably data structures).
For example, a stack or queue. You can easily implement this in JavaScript/TypeScript:

```ts
class Stack {
  private array: any[] = [];
  
  constructor(...items: any[]) {
    this.array.push(...items);
  }
  
  push(item: any) {
    return this.array.push(item);
  }
  
  pop() {
    return this.array.pop();
  }
}
```

Unfortunately, we have almost no choice but to use a taboo type like `any` (`unknown` will be very annoying to work with).

We *could* make a specialized class only for numbers, strings, booleans, etc, but that's very inefficient, and how would the user define their own classes without creating one from scratch?
We can't make a factory function either... that would require generics.

Which is why generics exist (no, not so we can make factory functions); so we can *generalize* something, whether it be a class, function, or interface.

The syntax for defining generics is much like Java. <>

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
