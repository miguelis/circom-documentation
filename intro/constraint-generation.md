# Constraint Generation

Circom allows programmers to define the constraints that define the arithmetic circuit. All constraints must be of the form A\*B + C = 0, where A, B and C are linear combinations of signals. Circom will apply some minor transformations on the defined constraints in order to meet the format A\*B + C = 0:

* Moves from one side of the equality to the other.
* Applications of commutativity of addition.
* Multiplication \(or divisions\) by constants.

A constraint is imposed with the operator ===, which creates the simplified form of the given equality constraint.

```text
a*(a-1) === 0;
```

Which is equivalent to:

```text
out === 1 – a*b;
out <-- 1 - a*b;
```

As mentioned before, assigning a value to a signal using &lt;-- and --&gt; is considered dangerous and should, in general, be combined with adding constraints with ===, which describe by means of constraints which are the assigned values. For example:

```text
a <-- b/c;
a*c === b;
```

In the constructive phase, a var can contain arithmetic expressions that are built using multiplication, addition, and other variables or signals and field values. Only quadratic expressions are allowed to be included in constraints. Other arithmetic expressions beyond quadratic or using other arithmetic operators like division or power are not allowed as constraints. To understand the constructive part of Circom, we need to consider the following type of expressions:

* Constant values: only a constant value is allowed.
* Linear expression: an expression where only addition is used. It can also be written using multiplication of variables by constants. For instance, the expression 2\*x + 3\*y + 2 is allowed, as it is equivalent to x + x + y + y + y + 2. 
* Quadratic expression: it is obtained by allowing a multiplication between two linear expressions and addition of a linear expression: A\*B - C, where A, B and C are linear expressions. For instance, \(2\*x + 3\*y + 2\) \* \(x+y\) + 6\*x + y – 2.
* Non quadratic expressions: any arithmetic expression which is not of the previous kind.

At the level of the generated wasm code, the var has a different interpretation when they only contain field elements. Hence, expressions are only present in one of the two phases of Circom.

The following example shows the generation of expressions:

```text
 signal input a;
 signal output b;
 var x = a*a;
 x += 3;
 b <== x;
```

