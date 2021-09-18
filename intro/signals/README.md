# Signals

The arithmetic circuits built with Circom operate on signals, which contain field elements in Z/pZ. Signals can be named with an identifier or can be stored in arrays and are declared using the keyword signal as in the examples below. Signals can be defined as input or output, and are considered intermediate signals otherwise. Signals are always considered private. The programmer can distinguish between public and private signals only when defining the main component, by providing the list of public input signals \(see below\). From the programmer's point of view, only input and output signals are visible from outside the circuit, and hence no intermediate signal can be accessed.

```text
signal input in;
signal output out[n];
signal input a;
```

Signals are immutable, which means that once they have a value assigned, this value cannot be changed anymore. Hence, if a signal is assigned twice, a compilation error is generated.

At compilation time, the content of a signal is always considered unknown \(see [Unknowns](../unknowns.md)\), even if a constant is already assigned to them. The reason for that is to provide a precise \(decidable\) definition of which constructions are allowed and which are not, without depending on the power of the compiler to detect whether a signal has always a constant value or not.

Signals can only be assigned using the operations &lt;-- or &lt;== \(see [Basic operators](../common-programming-concepts/basic-operators.md)\) with the signal on the left hand side and and --&gt; or ==&gt; \(see [Basic operators](../common-programming-concepts/basic-operators.md)\) with the signal on the right hand side. The safe options are &lt;== and ==&gt;, since they assign values and also generate constraints at the same time. Using &lt;-- and --&gt; is, in general, dangerous and should only be used when the assigned expression can not be included in a constraint, like in the following example.

```text
out[k] <-- (in >> k) & 1;
```

