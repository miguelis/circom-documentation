# Data Types

The basic var types in Circom are:

* Field element values: integer values modulo the prime number p \(see [Signals](../signals/)\). This is the default type for all signals and basic variables.
* Arrays: they can hold a finite number of elements \(known at compilation time\) of the same type \(signal, var, or the same type of components or arrays again\). The elements are numbered from zero on and can be accessed using the corresponding index of their position. Array access is made using square brackets. Declaration of an array of a given type is made by adding \[\] aside of the variable identifier and including the size between the brackets \(which should be defined using constant values and/or numeric parameters of templates\).

The notation m\[i,j\] for arrays of arrays \(matrices\) is not allowed. The access and the declaration should be consistent with their type and hence we access and declare with m\[i\]\[j\], since m\[i\] is an array. Examples of declarations with and without initialization:

```text
var x[3] = [2,8,4];
var z[n+1];  // where n is a parameter of a template
var dbl[16][2] = base;
var y[5] = someFunction(n);
```

On the other hand, the following case will produce a compilation error, since the size of the array should be explicitly given;

```text
var z = [2,8,4];
```

Finally, the type of signals needs to be declared as they cannot be assigned globally as an array. They are assigned by positions.

