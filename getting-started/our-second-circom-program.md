# Our second circom program \(done\)

circom allows programmers to define the [constraints](../intro/constraint-generation.md) that define the arithmetic circuit. All constraints must be of the form A\*B + C = 0, where A, B and C are linear combinations of signals. More details about these equations can be found [here](../intro/constraint-generation.md).

## BinaryCheck

Let us build a circuit that checks if the input signal is binary. In case it is, the circuit returns an ouput signal which is tagged as [Binary](../intro/signals/tags.md) with the same value than`in`.

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

After declaring the signals of the circuit, we use operator `===`to introduce the constraint $$in * (in -1) = 0$$. The solutions of this constraint are $$in = 0$$ and $$in = 1$$. This means that the constraint has solution if and only if the input signal is binary.

Instruction `out <== in` does not only assign the value of signal `in` to signal `out`, but it also adds the constraint $$out = in$$ to the set of constraints that define the circuit. Then, when both constraints have solution, it is guaranteed that the output signal is binary.

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
   component comp = Multiplier2();
   component binCheck[2];

   //Statements.
   binCheck[0] = binaryCheck();
   binCheck[0].in <== in1;
   binCheck[1] = binaryCheck();
   binCheck[1].in <== in2;
   comp.in1 <== binCheck[0].out;
   comp.in2 <== binCheck[1].out;
   out <== comp.out;
}
```

Simplifying, the 2-gate AND circuit can be defined by the next constraints:

$$in1 * (in1 -1) = 0, ~in2*(in2-1)=0,~ out=in1*in2$$

These constraints are satisfiable if and only if `in1, in2` are binary signals. Consequently, `out` will also be binary.

## An N-gate logic AND

Finally, let us build an N-gate logic AND using circuit `Multiplier2` and `binaryCheck`.

```text
template AndN (N){
   //Declaration of signals and components.
   signal:Binary input in[N];
   signal:Binary output out;
   component compAnd[N-1];
   component compBin[N];

   //Statements.
   for(var i = 0; i < N; i++){
          compBin[i] = binaryCheck();
         compBin[i].in <== in[i];
   }
   for(var i = 0; i < N-1; i++){
          compAnd[i] = Multiplier2();
   }
   compAnd[0].in1 <== compBin[0].out;
   compAnd[0].in2 <== compBin[1].out;
   for(var i = 0; i < N-2; i++){
       compAnd[i+1].in1 <== compAnd[i].out;
       compAnd[i+1].in2 <== compBin[i+2].out;

   }
   out <== compAnd[N-2].out; 
}
```

This program is very similar to `MultiplierN`, but every signal involved in itis binary.

It is important to highlight that we cannot use a \(2N-1\)-dimensional array to instantiate all the components since, every component of an array must be an instance of the same template with \(optionally\) different parameters.

This code can be found [here](https://github.com/miguelis/circom-handbook/blob/main/circom-examples/N_gate_AND.circom).

