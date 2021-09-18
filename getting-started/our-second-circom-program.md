---
description: >-
  This tutorial guide you through the process of writing your second circom
  program using other important features of circom: constraints, signal tags...
---

# Our second circom program

circom allows programmers to define the [constraints](../intro/common-programming-concepts/constraint-generation.md) that define the arithmetic circuit. All constraints must be of the form A\*B + C = 0, where A, B and C are linear combinations of signals. More details about these equations can be found [here](../intro/common-programming-concepts/constraint-generation.md). 

## BinaryCheck 

Let us build a circuit that checks if the input signal is binary. In case it is, the circuit returns an output signal which is tagged as [Binary]() with the same value than`in`. 

```text
template binaryCheck () {
   //Declaration of signals.
   signal input in;
   signal:Binary output out;
   
   //Statements.
   in * (in-1) === 0;
   out <== in;
}
```

After declaring the signals of the circuit, we use the operator `===`to introduce the constraint $$in * (in -1) = 0$$. The solutions of this constraint are $$in = 0$$ and $$in = 1$$. This means that the constraint has solution if and only if the input signal is binary.

The instruction `out <== in` not only assigns the value of signal `in` to signal `out`, but it also adds the constraint $$out = in$$ to the set of constraints that define the circuit. Then, when both constraints have solution, it is guaranteed that the output signal is binary. Sometimes, we only want to assign the value of a signal but not adding the corresponding constraint. In this case, we will use the operator `<--` and `-->`. The differences between `<--/-->` and `<==/==>` are described [here](../intro/common-programming-concepts/signals/).

## A 2-gate logic AND

We are going to use the circuits `Multiplier2` and `binaryCheck` to build a 2-gate logic AND.

```text
template Multiplier2(){
   //Declaration of signals
   signal input in1;
   signal input in2;
   signal output out;
   
   //Statements.
   out <== in1 * in2;
}

template binaryCheck () {
   //Declaration of signals.
   signal input in;
   signal:Binary output out;
   
   //Statements.
   in * (in-1) === 0;
   out <== in;
}

template And2(){
   //Declaration of signals and components.
   signal:Binary input in1;
   signal:Binary input in2;
   signal:Binary output out;
   component mult = Multiplier2();
   component binCheck[2];
   
   //Statements.
   binCheck[0] = binaryCheck();
   binCheck[0].in <== in1;
   binCheck[1] = binaryCheck();
   binCheck[1].in <== in2;
   mult.in1 <== binCheck[0].out;
   mult.in2 <== binCheck[1].out;
   out <== mult.out;
}
```

Simplifying,  the 2-gate AND circuit can be defined by the next constraints:

$$in1 * (in1 -1) = 0, ~in2*(in2-1)=0,~ out=in1*in2$$ 

These constraints are satisfiable if and only  if `in1, in2` are binary signals. Consequently, `out` will also be binary.

## An N-gate logic AND

Finally, let us build an N-gate logic AND using circuit `Multiplier2` and `binaryCheck`.

```text
template AndN (N){
   //Declaration of signals and components.
   signal:Binary input in[N];
   signal:Binary output out;
   component mult[N-1];
   component binCheck[N];
   
   //Statements.
   for(var i = 0; i < N; i++){
   	   binCheck[i] = binaryCheck();
	     binCheck[i].in <== in[i];
   }
   for(var i = 0; i < N-1; i++){
   	   mult[i] = Multiplier2();
   }
   mult[0].in1 <== binCheck[0].out;
   mult[0].in2 <== binCheck[1].out;
   for(var i = 0; i < N-2; i++){
	   mult[i+1].in1 <== mult[i].out;
	   mult[i+1].in2 <== binCheck[i+2].out;
   	   
   }
   out <== mult[N-2].out; 
}
```

This program is very similar to `MultiplierN`, but every  signal involved in it is binary.

It is important to highlight that we cannot use a \(2N-1\)-dimensional array to instantiate all the components since, every component of an array must be an instance of the same template with \(optionally\) different parameters.

This code can be found [here](https://github.com/miguelis/circom-handbook/blob/main/circom-examples/N_gate_AND.circom).

