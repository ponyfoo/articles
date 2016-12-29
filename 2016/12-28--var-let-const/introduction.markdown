A `let` statement indicates that a variable can't be used before its declaration, due to the Temporal Dead Zone rule. This isn't a convention, it is a fact: if we tried accessing the variable before its declaration statement was reached, the program would fail. These statements are block-scoped and not function-scoped; this means we need to read less code in order to fully grasp how a `let` variable is used.

The `const` statement is block-scoped as well, and it follows TDZ semantics too. The upside is that `const` bindings can only be assigned during declaration.

Note that this means that the variable binding can't change, but it doesn't mean that the value itself is immutable or constant in any way. A `const` binding that references an object can't later reference a different value, but the underlying object can indeed mutate.

In addition to the signals offered by `let`, the `const` keyword indicates that a variable binding can't be reassigned. This is a strong signal. You know what the value is going to be; you know that the binding can't be accessed outside of its immediately containing block, due to block scoping; and you know that the binding is never accessed before declaration, because of TDZ semantics.

Constraints such as those offered by `let` and `const` are a powerful way of making code easier to understand. Try to accrue as many of these constraints as possible in the code you write. The more declarative constraints that limit what a piece of code could mean, the easier and faster it is for humans to read, parse, and understand a piece of code in the future.
