---
description: >-
  This tutorial guide you through the process of writing your first program
  using the main features of circom: signals, variables, templates, components,
  and arrays.
---

# Our first circom program

## A 2-input Multiplier

The arithmetic circuits built using circom operate on signals. Let us build our first circuit: a 2-input multiplier.

```text
/*This circuit multiplies in1 and in2.*/
template Multiplier2 () {
   //Declaration of signals.
   signal input in1;
   signal input in2;
   signal output out;
   
   //Statements.
   out <== in1 * in2;
}
```

First, we use the reserved keyword `template`to define our new circuit, called `Multiplier2`.  Now, we have to define its [signals](../intro/common-programming-concepts/signals/). Signals can be named with an identifier, e.g.,  `in1, in2, out.`  In this circuit, we have two input signals`in1, in2` and an output signal `out`.  Finally, we use `<==` to set that the value of `out` is the result of multiplying the values of `in1` and `in2`.  Equivalently, we could have also used the operator `==>`, e.g., `in1 * in2 ==> out`.

Let us notice that in each circuit, we first declare its signals, and after that, the assignments to set the value of the output signals.

## A 3-input Multiplier

Building on top of the 2-input multiplier, we can build a 3-input multiplier.

```text
//This circuit multiplies in1, in2, and in3.
template Multiplier3 () {
   //Declaration of signals and components.
   signal input in1;
   signal input in2;
   signal input in3;
   signal output out;
   component mult1 = Multiplier2();
   component mult2 = Multiplier2();

   //Statements.
   mult1.in1 <== in1;
   mult1.in2 <== in2;
   mult2.in1 <== mult1.out;
   mult2.in2 <== in3;
   out <== mult2.out;
}
```

As expected, we first declare three input signals `in1, in2, in3,` and an output signal `out` and two instances of `Multiplier2` . Instantiations of templates are done using the keyword `component`. We need an instance `mult1` to multiply `in1` and `in2`. In order to assign the values of the input signals of `mult1` we use the dot notation `"."`. Once `mult1.in1` and `mult1.in2` have their values set, then the value of `mult1.out` is computed. This value can be now used to set the input value of `mult2`  of the second instance of `Multiplier2`to multiply `in1*in2` and `in3` obtaining the final result  `in1*in2*in3`.

Finally, every execution starts from an initial [main component](../intro/common-programming-concepts/template/the-main-component.md) defined as follows.

```text
component main {public [in1,in2,in3]} = Multiplier3();
```

Here, we indicate that the initial component for our first circom program is the circuit `Multiplier3` which has three public signals: `in1, in2` and `in3`.

This program can be found [here](https://github.com/miguelis/circom-handbook/blob/main/circom-examples/multiplierN.circom).

## An N-input Multiplier

When defining a template, we can use [parameters](../intro/common-programming-concepts/template/) to build generic circuits. These parameters must have a [known](../circom-insight/circom-compiler/unknowns.md) value at the moment of the instantiation of the template. Following up the previous example, we can implement an N-input multiplier, where `N` is a parameter.

```text
template MultiplierN (N){
   //Declaration of signals and components.
   signal input in[N];
   signal output out;
   component comp[N-1];
   
   //Statements.
   for(var i = 0; i < N-1; i++){
   	   comp[i] = Multiplier2();
   }
```

In addition to the parameter`N`, two well-known concepts appear in this fragment of code: [arrays](../intro/common-programming-concepts/data-types.md) and [integer variables](../intro/common-programming-concepts/data-types.md). 

As we have seen for a 3-input multiplier, we need 3 input signals and 2 components of `Multiplier2`. Then, for an N-input multiplier, we need an N-dimensional array of input signals and an \(N-1\)-dimensional array of components of `Multiplier2`. 

We also need an integer variable `i` to instantiate each component `comp[i]`. Once this is done, we have to set the signals for each component as follows:

```text
   comp[0].in1 <== in[0];
   comp[0].in2 <== in[1];
   for(var i = 0; i < N-2; i++){
	   comp[i+1].in1 <== comp[i].out;
	   comp[i+1].in2 <== in[i+2]; 
   }
   out <== comp[N-2].out; 
}
```

Similarly to `Multiplier3`, each output signal of a component becomes one of the input signals of the next component. Finally, `out` is set as the output signal of the last component and its value will be `in[0]*in[1]*...*in[N-1]`. Finally, we define as main component a `MultiplierN` with `N = 3`.

```text
component main {public [in1,in2,in3]} = MultiplierN(3);
```

This code can be found [here](https://github.com/miguelis/circom-handbook/blob/main/circom-examples/multiplierN.circom).

```text
template Multiplier2(){
   //Declaration of signals.
   signal input in1;
   signal input in2;
   signal output out;
   
   //Statements.
   out <== in1 * in2;
}

template Multiplier3 () {
   //Declaration of signals.
   signal input in1;
   signal input in2;
   signal input in3;
   signal output out;
   component mult1 = Multiplier2();
   component mult2 = Multiplier2();
   
   //Statements.
   mult1.in1 <== in1;
   mult1.in2 <== in2;
   mult2.in1 <== mult1.out;
   mult2.in2 <== in3;
   out <== mult2.out;
}

template MultiplierN (N){
   //Declaration of signals.
   signal input in[N];
   signal output out;
   component comp[N-1];
 
   //Statements.
   for(var i = 0; i < N-1; i++){
   	   comp[i] = Multiplier2();
   }
   comp[0].in1 <== in[0];
   comp[0].in2 <== in[1];
   for(var i = 0; i < N-2; i++){
	   comp[i+1].in1 <== comp[i].out;
	   comp[i+1].in2 <== in[i+2];
   	   
   }
   out <== comp[N-2].out; 
}

component main {public [in1,in2,in3]} = MultiplierN(3);
```

