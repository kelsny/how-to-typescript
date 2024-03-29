### Archived in favor of https://type-level-typescript.com/

English · [中文](./translations/zh)

Formal · [Informal](./styles/informal) · [Uncensored](./styles/uncensored)

# How to TypeScript

> People always ask, "How do you do your type magic stuff?", so I'm going to try and answer them.

[Ew skip the boring sh\*t](#part-1)

#### Prerequisites

In this book I'm going to assume you know the following:

-   [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) - Fundamentals, ES6 features, and the standard library.
-   [TypeScript](http://www.typescriptlang.org/) - What it is, why use it, basic syntax.
-   [Node.js](https://nodejs.org/en/) - What it is, `global` and `process`, native modules' API.

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
const primitive = "hello world"; // string

const boxed = new String("hello world"); // String

const constructor = String; // StringConstructor
```

Keep in mind that there is a _really_ big difference between a primitive, boxed primitive, and the boxed primitive constructor's types.
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
    };
};
```

There is also the `object` type, which is also pretty taboo like `any`, because most things can be assigned to it.
Another one like `object` is `{}`. And also like primitives, there is `Object` and `ObjectConstructor`.

You _might_ yse `ObjectConstructor` because of `Object`'s built-in methods, for example, `Object.entries` (in types, `ObjectConstructor["entries"]`).

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

type FooAndBar = Foo & Bar; // { foo: string; bar: string }

type ImpossibleIntersection = string & number; // is never
```

**Type of something (`typeof`)\***

Ok, this one's like the JavaScript `typeof` operator, but it retrieves the inferred or explicitly defined type of a value.

```ts
import Module from "library";

type TypeOfImportedThing = typeof Module;
```

**Is subtype (`extends`)**

This one isn't the `extends` like you use in a class, although it does relate to it in a way. No, `extends` can be used in one of 2 ways:

-   As a generic type constraint (you will see what I mean)
-   As a way to check if a type extends another type (if a type is a subtype of another type)

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
type AliasForStringArray = string[];
```

Multi-dimensional

```ts
type Array2D = number[][];
```

Now for something that might blow you statically typed compiled language people's minds.

It's important to think about `[]` as an _operator_. It takes its operand on the left and wraps it as an array type. In this way, we can compose complex array types:

```ts
type AReallyComplexArrayType = { key: ((string | number)[] & { depth: number })[] }[];
```

Note the use of parentheses; `(string | number)[]` is definitely not the same as `string | number[]`.

Tuples are declared in a different fashion. They look like array literals in code, but instead of values, it's types in their place.

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

Oh another weird symbol? What's _this_ one for?
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

We _could_ make a specialized class only for numbers, strings, booleans, etc, but that's very inefficient, and how would the user define their own classes without creating one from scratch?
We can't make a factory function either... that would require generics.

Which is why generics exist (no, not so we can make factory functions); so we can _generalize_ something, whether it be a class, function, or interface.

The syntax for defining generics is much like Java. Use angle brackets, with the generic parameters in between them (TypeSript does not have the [diamond operator](https://en.wikipedia.org/wiki/Generics_in_Java#Diamond_operator)).

```ts
class Stack<T> {
    private array: T[] = [];

    constructor(...items: T[]) {
        this.array.push(...items);
    }

    push(item: T) {
        return this.array.push(item);
    }

    pop() {
        return this.array.pop();
    }
}
```

Just like that we have a class that can now support any type.
And we can also restrict the objects that this class will hold:

```ts
class Stack<T extends { id: number; }> {
  private array: T[] = [];

  ...
}
```

Let's take a look at another common example: the hashmap.
Maps hold keys that _map_ to a value.
In Java, we have `Map<K, V>`, where `K` is the key's type and `V` is the value's type.
TypeScript is exactly the same:

```ts
const map = new Map<string, number>([["key", 123]]);

map.get("key"); // returns number
```

Generics also apply to interfaces:

```ts
interface StackLike<T> {
    push(item: T): number;
    pop(): T;
}
```

And of course, we can still use `extends` to constrain the types:

```ts
interface StackLike<T extends { id: number }> {
    push(item: T): number;
    pop(): T;
}
```

Lastly, generics also apply to functions.
I'll only give the basic syntax for defining a generic function.
The rest I leave to you, to experiment and discover the quirks and syntax for overloads.

```ts
function genericFn<T, Foo extends Constraint>(arg1: T, arg2: Foo): T | Foo { ... }
```

## Chapter 4 - Sprinting

The last chapter of review. What shall I cover here?
I chose extra things that aren't necessary, but could help you debug or develop faster.
Let's get started right away.

#### The `declare` keyword

When you're rapidly prototyping types, you might have to declare a few types, variables, or functions so you can properly test your types.
However, some implementations or complex structures for variables might slow you down. What can you do to prevent this?
Declaring _only_ their types, so you do not have to spend time on actually making them work (i.e. don't have to write code to make the function do something).

This is possible with the `declare` keyword.
`declare` is a special keyword with many uses, including [Module Augmentation](typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation), but here we'll use it as a great way to tell TypeScript that some things are defined.

Firstly, let's use it to easily define a variable of a complex type.

```ts
type MightBeNested<T> = (T | MightBeNested<T>)[];
```

Here we have a complex type, a "might-be-nested" array of "might-be-nested" arrays.
You could define a variable with filler/dummy data, but that's slow and inefficient.
Instead, we can use `declare` to just declare that there is a variable of this type.

```ts
declare const augmentedVariable: MightBeNested<number>;

console.log(augmentedVariabled); // in runtime (strict mode), this will give an error since augmentedVariable is not actually defined
```

There is no variable defined here, but since we have used `declare`, TypeScript _thinks_ there is a variable named `augmentedVariable`.

Extending this to functions:

```ts
declare function complexFn(arg1: Foo, arg2: Bar): Baz;
```

See? Now you don't need to worry about the function's implementation so that the types match.

# Part 2 - The new horizon

## Chapter 5 - Look the other way

Congratulations! You've either made it here or skipped the first part!
Ok, ok, so you think you know TypeScript types fairly well, after all, you're practically following my every word.
Guess what? It was a prank! Happy April Fools, fool!

To properly learn the art of type magic/abuse, you have to think about types in a different way.
And that, much, much, much different way is to think of types just like code.
What do I mean by that?

Take a look:

```ts
type MightBeNested<T> = (T | MightBeNested<T>)[];

// think of this type as a function like this

function MightBeNested(T) {
  return (T | MightBeNested(T))[];
}
```

This "function" takes a parameter `T` and returns `(T | MightBeNested(T))[]`. In other words, it's **recursive**, just like the type.
In type magic, recursion is heavily used/optimized in the form of tail recursion (see: [TCO](https://en.wikipedia.org/wiki/Tail_call)).
When you abuse types, you should almost always think of them as a function, like this.
It even helps to write them first as a function (limited in syntax and features, of course), then convert it to a type.

## Chapter 6 - On one condition...

So far, pretty cool. Types can be thought of as simple recursive functions.

You've probably heard TypeScript's types are Turing complete, so... where's the Turing complete-ness?

**Enter: conditional types.**

Conditional types are very much like plain old ternary operators. They have three operands: a condition, type if true, type if false.
This is similar to a ternary, except it uses expressions instead of types.

Conditional types are always in the form `SubType extends SuperType ? TypeIfTrue : TypeIfFalse`.

As a simple example to get you comfortable with reading this, we'll make a simple type that gives us `true` if the parameter extends `string`, and false otherwise.
To start, create the type:

```ts
type IsString<T> = T;
```

Check if it extends `string`:

```ts
type IsString<T> = T extends string;
```

We're not done yet. Unlike normal JavaScript even if we want to return a boolean, we have to explicitly include the boolean values:

```ts
type IsString<T> = T extends string ? true : false;
```

Yes, I know, it's repetitive if you need to do this a lot. Hopefully someday this will be a feature of TypeScript (at least a compiler option).
And just like that, we've created our first conditional type, albeit much more useless than what conditional types can really do.
Let's test it now.

```ts
type T0 = IsString<string>; // true
type T1 = IsString<"yes i am a string">; // true
type T2 = IsString<number>; // false
type T3 = IsString<123>; // false
```

Nice, seems to work!

Since conditional types, are again, much like ternaries, you can chain/nest them.

Chaining:

```js
condition1 ? expr1 : condition2 ? expr2 : expr3;
```

Nesting:

```js
condition1 ? (condition2 ? expr1 : expr2) : expr3;
```

Or you can even do both at once (really hard to read, use if-elif-else instead).
Note that chaining represents an OR operator if you chain it like this:

```js
condition1 ? true ? condition2 ? true : false
```

Likewise, nesting represents an AND operator if you nest them like this:

```js
condition1 ? (condition2 ? true : false) : false;
```

Keep these in mind, since they will be extremely useful later, when we practice logic gates with conditional types.

## Chapter 7 - Template for rest

Template literals have probably been one of the most exciting and innovative features to come to JavaScript, along with the spread/rest operator.
Which is why TypeScript also supports them in types:

```ts
type UseBackticks = `hello world!
new lines
are supported, too\
 can also escape them`;

type MyArrayType = [1, 2, 3];

type CloneOfMyArrayTypeUsingTheSpreadOperator = [...MyArrayType];
```

Spread operators are used often in recursive types, since most recursive type abuse needs to accumulate data (side tangent: [Is Type Abuse a Good Way to Learn Functional Programming](./tangents/fp.md)). They're actually really intuitive to use, and I'm pretty sure you can do some digging yourself on this one.

Template literals however, are a big mess. Not in the sense of inconsistency and unintuitive-ness, but in the grander sense that it has so many different uses and features.
If I were to write an article about template literals, it would probably be longer than all volumes of Harry Potter combined, and in fact, there is an article about template literals... not that long but still extremely long.
That's why I'll only cover the things you absolutely need to know to get started with abusing types.

First, the actual templating part. In the first example, I only used backticks to replace quotes and newlines, which I've got to admit, not a fair and effective usage of them.
To not be really confusing when introducing this, I'll only put type literals inside templates, so it looks just like JavaScript, like so:

```ts
type Hello123 = `Hello${123}`;
```

Wonderful. Works _just_ like regular template literals. Same goes for any type that can be interpolated with a string (_cough_ symbols _cough_).
Ok, now for something that blew my mind at first... putting a f\*cking type in there instead (and yes, this is where I curse a lot more).

```ts
type HelloWithAnyNumber = `Hello${number}`;
```

Right. Crazy. But what does this actually do? Well it allows any string that starts with `"Hello"` and ends with a number.

Some examples:

-   `Hello123`
-   `Hello0`
-   `Hello0.0`
-   `Hello0.123`
-   `Hello123.123`

What if I but `string` instead of `number`?
You guessed it. I can put any string after the `Hello`:

-   `HelloWorld`
-   `Hello world`
-   `Hello`

Make sure you see why these work.
Now for boolean, right, you can only have `Hellotrue` or `Hellofalse`.

Ok, interesting. What about... a union of types...

Let's first try `string | number`.

Hmm, it appears now, I can put any string or number after `Hello`.

Pretty much expected, if you ask me. What's more interesting is putting a union of literal types.

For example:

```ts
type HelloPeople = `Hello ${"Alice" | "Bob" | "Charlie"}`;
```

What's `HelloPeople`?

I'll wait until your mouth is not open anymore.

...

Yes! I know! TypeScript automagically created a union of `Hello Alice`, `Hello Bob`, and `Hello Charlie`.

This is an important pattern where you can use TypeScript to generate similar string unions for you.

Fireship has made an amazing YouTube short detailing this [here](https://www.youtube.com/watch?v=5JqzCjg4YRU) (must watch).

Insane. You would never fathom that this would be possible in TypeScript before template literals.

Now why is this useful? Because you can limit what your endusers input into your library or API.
TypeScript is all about type safety, so we should be able to create custom string types to suit our design decisions.

Let's use this to create a type that checks if a string is numerical (i.e. checks if the string can be turned into a number using `Number`).

```ts
type IsNumerical<S extends string> = S extends `${bigint}` ? true : false;
```

Here, we use `extends string` to make sure `S` is a string. Then we use a conditional type along with a template literal that includes a `bigint`.
Time to test!

```ts
type T0 = IsNumerical<"123">;
type T1 = IsNumerical<"123.123">;
type T2 = IsNumerical<123>;
type T3 = IsNumerical<true>;
```

Pretty nice, if I do say so myself (and I do).

## Chapter 8 - Make Inferences

TypeScript has conditional operators, which already allow us to do many, many cursed things. For example, you can make it do basic arithmetic (we will make this in the next chapter).
But for TypeScript's type system to be even more useful, it should be able to be told when to infer and what to do with the inferred type.
For this functionality, there is the `infer` keyword.

The `infer` keyword does exactly what it sounds like. It infers types. As a really simple example, let's recreate the utility type `ReturnType`.

First, define the type:

```ts
type ReturnType<F> = F;
```

Constrain `F` to only functions:

```ts
type ReturnType<F extends (...args: any[]) => any> = F;
```

We'll use `any` here because it can represent any type, which we want, in this case. Now... we'll use a conditional type.

```ts
type ReturnType<F extends (...args: any[]) => any> = F extends (...args: any[]) => any ? F : F;
```

We notice that `F extends (...args: any[]) => any` will always be false (since `F` is already constrained to that type), so we put `never` as the type if false:

```ts
type ReturnType<F extends (...args: any[]) => any> = F extends (...args: any[]) => any ? F : never;
```

But how do we infer the return type? It's annotated as `any` right now.
With the `infer` keyword, of course. It always goes like this: `infer Identifier`, where `Identifier` is the inferred type.

```ts
type ReturnType<F extends (...args: any[]) => any> = F extends (...args: any[]) => infer R ? F : never;
```

Like this, TypeScript will say, "Hey, there's something I need to infer. I'll store what I think in `R`." Now we can use `R`, the result of TypeScript's inference.
Note that you can only used the identifier only after the `?`. It will not be defined in the "type-if-false" clause.
For example, this is not correct:

```ts
type ReturnType<F extends (...args: any[]) => any> = F extends (...args: any[]) => infer R ? F : R;
```

`R` is not defined after the `:`, so this gives us an error.

However, this is correct, syntactically and logically (what we want):

```ts
type ReturnType<F extends (...args: any[]) => any> = F extends (...args: any[]) => infer R ? R : never;
```

You can read this as English if you want. "If F extends a function, infer its return type, otherwise, `never`".

Since the inferred type is now an identifier we can use... We can check if it extends another thing as well.
Let's use this to create a type, `ReturnTypeExtendsFoo`, which will check if the function's return type extends, well, `Foo`.

```ts
type ReturnTypeExtendsFoo<F extends (...args: any[]) => any> = F extends (...args: any[]) => infer R ? (R extends Foo ? true : false) : false;
```

Now let's read this again. "If F extends a function, infer its return type. If the inferred return type extends Foo, true, otherwise, false. If F is not even a function, false."
Wait a minute... This kinda sounds like `ReturnType` with an extra step after we infer the return type. Well yes, because you can also express this type as

```
type ReturnTypeExtendsFoo<F extends (...args: any[]) => any> = ReturnType<F> extends Foo ? true : false;
```

I just wanted to show you that the inferred type can be used just like a generic parameter :)

Alright so we can infer function return types... Can we do parameters? Why yes, of course we can.
Can you guess what's the syntax to do so?

Hm, let's see... To make it work with functions, instead of putting a plain old type as the return type, we slapped in `infer SomeIdentifier` instead.
In function parameter type annotations, the syntax is `(arg0: Type, arg1: Type)` or `(...args: SomeTupleOrArrayType)`. Let's try this again.
We'll replace the type with `infer SomeIdentifier` again.

```ts
type GimmeYourParameters<F extends (...args: any[]) => any> = F extends (...args: infer ArgTypes) => any ? ArgTypes : never;
```

Wow! It actually worked! As you can probably guess, it works the same for singular parameters.

```ts
type GimmeYourFirstParameter<F extends (...args: any[]) => any> = F extends (arg: infer ArgType) => any ? ArgType : never;
```

Nice! But hey, what's the fun in _only_ inferring types of functions? Fortunately, we can do the same to strings and arrays.
Let's talk about strings first, since they might have a little more magic than you may think (arrays do, too, but I think their magic isn't as handy :D).

## Chapter 9 - Abusing inferences

Alright, out with it, what do you think we can do with `infer`? What's the most cursed, abusive, and imaginative way, you can _possibly_ use `infer`?

To me, the first thing that comes to mind, are parsers. Made entirely out of TypeScript types, and only out of TypeScript types. Literal, f\*cking parsers.
You name it. JSON parsers, YAML parsers, Markdown parsers, DSL parsers, text parsers, CSV parsers... Anything that can be parsed, can be parsed in TypeScript types.

So hopefully that's got you a little excited about `infer`. Now let's actually see _how_ you can do the same.

Let's say we want to get a set of all the characters in a string. In plain JS, we'd probably do something like this:

```js
function GetChars(string) {
    return [...new Set(string.split(""))];
}
```

And, since a union of the same type evaluates to that type,

```ts
type UnionActsLikeASet = "foo" | "bar" | "foo"; // => "foo" | "bar"
```

unions act like a set, in a way.

Now, let's go back to template literals. Notice that we can do something like this:

```ts
type ABC = "a" | "b" | "c";

type StartsWithABC<S extends string> = S extends `${ABC}${string}` ? true : false;
```

Ok, if you remember correctly, `infer` is like a drop-in replacement for a type we want to infer...
And guess what, it works exactly the same here as well! Props to the designers of TypeScript for making `infer` pretty simple to use across many situations!

```ts
type ABC = "a" | "b" | "c";

type StartsWithABC<S extends string> = S extends `${ABC}${string}` ? (S extends `${infer Letter}${string}` ? Letter : never) : false;
```

Woah, not only can we tell if a string starts with a, b, or c, we can actually tell _which_ one it was!
Notice that I have put `never` in there, since `Letter` will _never_ not be inferred, since S must start with a, b, or, c.
Nice, nice, we can do some magic now. We can infer a letter in a string which is pretty cool, so now let's use it to get a set of all the characters in a string.

```ts
type GetChars<S extends string> = S extends `${infer Letter}${infer Rest}` ? ... : ...;
```

Ayo! What's this new fangled thing here? TWO `infer`'s? Ah don't worry.
When it's used like this, `Letter` will be a single character, and `Rest` will be the rest of the string.
This way, there is no ambiguity involved and it flows pretty smoothly.

We've got the first character and rest of the string now, what do we do with them?
We want a set, so let's put a union of Letter in:

```ts
type GetChars<S extends string> = S extends `${infer Letter}${infer Rest}` ? Letter | ... : ...;
```

What should the operands be? Well, since `GetChars` will evaluate to a union of letters, we can use this property, and stick it in like so:

```ts
type GetChars<S extends string> = S extends `${infer Letter}${infer Rest}` ? Letter | GetChars<Rest> : ...;
```

Note that it is `GetChars<Rest>`, not `GetChars<S>`. We want to get the characters for the rest of the string, not the current one.
Otherwise, it'll go into an infinite loop!

Hold your horses! What on earth goes at the end?
Well, following the earlier logic of no ambiguity, if the string does not extend `${infer Letter}${infer Rest}`, that must mean the string is empty.
If the string has one character, `Letter` will take on that character and `Rest` will be empty.
Knowing this, we can put `never` there, since `never | AnythingElse` evaluates to `AnythingElse` (check it yourself!).

```ts
type GetChars<S extends string> = S extends `${infer Letter}${infer Rest}` ? Letter | GetChars<Rest> : never;
```

And that's `GetChars` done! Play with it in the playground for a while, experiment with different strings and implementations.
You'll probably find that large strings make it incredibly slow, and I'll explain why and how to solve this in Part 3.

Ok, before we move on to inferring tuples/arrays, let's take a moment to talk about how infer works in strings.
I had so much time on my hands, I made a simple graphic in ASCII:

```txt
`${infer T}${infer R}`
example string
_ --- T
 _____________ --- R

`${infer T}inbetween${infer R}`
example inbetween string
________ --- T
           R --- _______

`ahead${infer T}inbetween${infer R}`
ahead example inbetween string
     _________ --- T
                 R --- _______
```

That should explain most of it. If you are using more than two `infer`'s to parse something, these rules should still apply.
I think it's best for you to play around with this until you're familiar with some of the edge cases and behaviour dealing with `infer` in template literals.

Yay time for the last part of `infer` 👏! Now we'll discuss tuples and arrays. For the most part, it's pretty much the same.

Recall that using `infer`, you simply replace the type with `infer AnIdentifier`. So, how does that look in tuples and arrays?

```ts
type InferArray<A extends any[]> = A extends (infer T)[] ? T : never;
```

Well... that's a useless type, but you can still see how it can infer the type of the array.

That's pretty much it for arrays. Onto tuples (which is, safe to say, far mor exciting)!
If you're familiar with tuples, you probably know how to make one that contains one element:

```ts
[MyType];
```

You know what time it is? `infer` time! Drop that sucker in and it'll work straight away!

```ts
[infer T]
```

Right that's cool, what about variable length tuples:

```ts
[number, ...number[]]
```

Well... turns out it's exactly the same! Keep in mind that the type to replace here is `number[]`, not `number`!

```ts
[number, ...infer Rest]
```

Wow! So simple!

And that's it for arrays. Nothing too complicated.
But you can take `infer` to greater heights.

You can, for example, literally use `infer` anywhere in place of a type... including object property types.

```ts
type InferProp<O> = O extend { foo: infer Bar } ? Bar : never;
```

I'll leave you with this new information to play around with! It's a lot to take in if you aren't used to it.

# Part 3 - Design & Develop

Hey, what's the use of learning this, if there's nothing to... use it _for_?

I saved this part till the end, because I believe you'd have a better time understanding the pattern and why it works, if you know each component that helped brought it to life first.
A little like how you learnt design patterns after you found out that your spaghetti code was spaghetti :)

## What's in a name

You're familiar with TypeScript now, so you understand that this:

```ts
type Foo = { bar: string };
```

is assignable to:

```ts
type Bar = { bar: string };
```

What are some use cases where we don't want this behaviour, and how would we fix it?

This is the nominal typing problem. Currently, TypeScript only works with structural types, i.e, types whose structures are the same (like `Foo` and `Bar`).
Nominal types come in handy, primarily for reducing typos/mix-ups with other types.
Personally, I've never had to use them, but if this pattern exists, I guess most people do.

The pattern is conceptually really simple: abuse the structural typing system.

Simply have your regular type, and intersect it with another type. This type could be an enum or another interface with an obscure property (or symbol as a property).

```ts
enum _WithEnum { _ = "" }

interface WithEnum_ {}

type WithEnum = WithEnum_ & _WithEnum;
```

```ts
interface _WithInterface { _withInterface = "" }

interface WithInterface_ {}

type WithInterface - WithInterface_ & _WithInterface;
```

Branding type: `_xxx`, regular type: `xxx_`.

And tada! You've got a nominal typing system. You can't assign anything to `WithEnum` or `WithInterface` except itself (or if user happens to use the property on `_WithInterface`...).

Nothing much here, although I think the enum style needs a little background.

Why is there a member of the enum? If there wasn't a member, then it would have been inferred as a numeric enum instead of a string enum, and a numeric enum intersected with a string is `never`.
Even though I'm not intersecting a string here, you should be careful!

## Practicing legal discrimination

Say you're implementing a protocol for some network, and each little "packet" contains data in JSON form.
This JSON structure varies depending on the packet type, which is defined in the JSON.

You could do:

```ts
interface Packet {
    type: string;
    [k: string]: string;
}
```

but that doesn't seem very safe or useful.

You try a union:

```ts
interface Packet {
    type: "foo" | "bar";
    [k: string]: string;
}
```

Slightly better, but still not good enough.

You could define all properties that can be used by the packet, and make them all optional, but then using the type would be a nightmare!
It'd just be a series of nightmarish if statements and assertions.

In desperation, you try a union type of all possible packet structures instead:

```ts
type Packet =
    | {
          type: "foo";
          foo: string;
      }
    | {
          type: "bar";
          bar: string;
      };
```

Then you stumble upon this:

```ts
if (packet.type === "foo") {
    packet; // { type: "foo"; foo: string; }
}
```

What? How? Well this is called a discriminated union, it works because TypeScript can narrow the union type to a single member because of that single common property.
Like the name suggests, it's simply a union of types that have a common property, the discriminative property, that allows TypeScript to differentiate one from the other.

Arguably, it's the most common and most useful.

## Disciplined

(Self-validating parameters)

```
// Common patterns amongst the type magic community
```

# Part 4 - Do It Yourself

```
// Walk-through a few projects and showcase a few
```

# Part 5 - Extensions

## Resources

- [Official TypeScript documentation](https://www.typescriptlang.org/docs/)

    A good reference to TypeScript's type system. Doesn't go into depth into type abuse however.
    
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

    Deeper dive into the semantics and behaviour of the type system.

### Afterword

Hi, thanks for taking a quick read of mine. Hopefully in this session you've been equipped with new knowledge, and now you
can do a little type magic yourself. If you have trouble with something about it, please don't hesitate to ask me on Discord!

Revision suggestions are welcome and any inaccuracies should be eliminated.

Thank you and good luck, have fun!

### Acknowledgements

-   [Me](https://github.com/cursorsdottsx/) - is that how this works?
-   [Cassie](https://github.com/BobobUnicorn/)
-   [Okku](https://github.com/0kku/)
-   [aero](https://github.com/dheerajpv/)
-   [Lebster](https://github.com/LebsterFace/)
-   Simon\_ - whoops I don't know their GitHub (yet)
-   StrangeQuarkAL - GitHub account not connected to Discord

### Contributing

**no.** this is my project and no one but me can contribute >:c

but you are of course free to suggest edits, content, and revisions!
