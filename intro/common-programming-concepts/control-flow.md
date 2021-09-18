# Control flow

We have standard constructions for defining the control flow of the program:

## Conditional statement: if-then-else

**if \( boolean\_condition \) block\_of\_code else block\_of\_code**

The else part is optional. If not included it means “else do nothing”.

```text
if (x >= 0) {
   x = y + 1;
   y += 1;
} else {
   x = c.b;
}
```

## Loop statement: for

**for \( initialization\_code ; boolean\_condition ; step\_code \) block\_of\_code**

If the initialization\_code includes a var declaration then its scope is reduced to the for statement and hence, using it later on \(without defining it again\) will produce a compilation error.

## Loop statement: while

**while \( boolean\_condition \) block\_of\_code**

It executes the block of code while the condition holds. The condition is checked every time before executing the block of code.

## Loop statement: do-while

**do block\_of\_code while \( boolean\_condition \)**

It executes the block of code until the condition fails. The condition is checked every time after executing the block of code.

**Important**: when constraints are generated in any block inside an if-then-else or loop statement, the condition must not be unknown \(see [Unknowns](../unknowns.md)\). This is because the constraint generation must be unique and cannot depend on unknown input signals. In case the expression in the condition is unknown and some constraint is generated, the compiler will generate an error message.

**Important**: Another compilation error is generated when the content of a var depends on some unknown condition: that is when the var takes its value inside an if-then-else or loop statement with an unknown condition. Then, the content of the variable is a non-quadratic expression and, as such, cannot be used in the generation of a constraint.

The control flow of the computations is like in other imperative languages, but the instantiation of components may not follow the sequential structure of the code as if they may depend on some input signal. The component instantiation will not be triggered until all input signals have a concrete value assigned.

