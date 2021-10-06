[Back to book](../README.md)

## Equality operators: Why I always use strict equal/not-equal

As a self-proclaimed seasoned TypeScript developer, I have a natural urge and sense of type safety.

"But cursors, you could've used `==` or `!=` to avoid manually type casting that operand", I hear you say.

Uh, did you even read how it compares/coerces its operands? No way, am I *ever* using that dumpster fire of inconsistency. 
I'd rather stick with manual, on-purpose type casting to maintain consistency and type error reports by TypeScript.

The equality operators use the [Abstract Equality Comparison Algorithm](https://262.ecma-international.org/5.1/#sec-11.9.3), which, already by its name, you can tell it's gonna be a hot mess of edge cases and unholy type coercion.

Contrast this to the [Strict Equality Comparison Algorithm](https://262.ecma-international.org/5.1/#sec-11.9.6). At least it makes sense for the most part!
And the part that doesn't make sense the most is explained by `NaN`'s definition and behaviour.

Furthermore, since I use TypeScript, using strict equality operators allows TypeScript to report an error if there is a mismatched type.

Ok, back to the book.
