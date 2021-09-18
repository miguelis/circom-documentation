# Signals & variables

The arithmetic circuits built using circom operate on signals, which contain field elements in Z/pZ. Signals can be named with an identifier or can be stored in arrays and declared using the keyword signal. Signals can be defined as input or output, and are considered intermediate signals otherwise. 

```text
signal input in;
signal output out[N];
signal inter;
```

This small example declares an input signal with identifier `in`, an N-dimension array of output signals with identifier `out` and an intermediate signal with identifier `inter`.

Signals are always considered private. The programmer can distinguish between public and private signals only when defining the main component, by providing the list of public input signals. 

```text
template Multiplier2(){
   //Declaration of signals
   signal input in1;
   signal input in2;
   signal output out;
   out <== in1 * in2;
}

component main {public [in1,in2]} = Multiplier2();
```

This example declares input signals `in1` and `in2` of the main component as public signals.

From the programmer's point of view, only input and output signals are visible from outside the circuit, and hence no intermediate signal can be accessed.

```text
template A(){
   signal input in;
   signal outA; //We do not declare it as output.
   outA <== in;
}

template B(){
   //Declaration of signals
   signal output out;
   component comp = A();
   out <== comp.outA;
}
```

This code produces a compilation error since signal `out2` is not declared as an output signal, then it cannot be assigned to signal `out`.

Signals are immutable, which means that once they have a value assigned, this value cannot be changed any more. Hence, if a signal is assigned twice, a compilation error is generated. This can be seen in the next example where signal `out` is assigned twice, producing a compilation error.

```text
template A(){
   signal input in;
   signal output outA; 
   outA <== in;
}

template B(){
   //Declaration of signals
   signal output out;
   out <== 0;
   component comp = A();
   out <== comp.outA;
}
```

At compilation time, the content of a signal is always considered unknown \(see [Unknowns](../../../circom-insight/circom-compiler/unknowns.md)\), even if a constant is already assigned to them. The reason for that is to provide a precise \(decidable\) definition of which constructions are allowed and which are not, without depending on the power of the compiler to detect whether a signal has always a constant value or not. 

```text
template A(){
   signal input in;
   signal output outA; 
   var i = 0; var out;
   while (i < in){
    out++; i++;
   }
   outA <== out;
}

template B(){
 component a = A();
 a.in <== 3;
}
```

This example produces a compilation error since value of signal `outA` depends on the value of signal `in`, even though, such a value is the constant 3.

Signals can only be assigned using the operations `<--` or `<==` \(see [Basic operators](../basic-operators.md)\) with the signal on the left hand side and and `-->` or `==>` \(see [Basic operators](../basic-operators.md)\) with the signal on the right hand side. The safe options are `<==` and `==>`, since they assign values and also generate constraints at the same time. Using `<--` and `-->` is, in general, dangerous and should only be used when the assigned expression cannot be included in a constraint, like in the following example.

```text
out[k] <-- (in >> k) & 1;
```



