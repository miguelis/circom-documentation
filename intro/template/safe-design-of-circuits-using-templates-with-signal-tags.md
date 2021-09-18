# Safe design of circuits using templates with signal tags

In this section, we show how signal tags can be used in library templates to safely build circuits. Here, since the compiler checks that all tags are met, it ensures that the user combines the templates appropriately. For instance, the following template for the AND gate requires the input signals to be Binary and states that the output is then Binary as well.

```text
template AND () {
   signal:Binary input in1;
   signal:Binary input in2;
   signal:Binary output out;

   out <== in1 * in2;
}
```

Note that no constraints stating that the signals are binary are introduced. However, the developer of the library is responsible to check on his own that the template is correct. That is, to prove that if the inputs are 0 or 1 then the output is 0 or 1 \(which is trivially true in this case\).

**It is very important to note that the compiler is neither checking that the signals are actually binary nor introducing constraints stating that. It is the responsibility of the programmer to be sure of that or impose the needed constraints.**

To ease the work of developers, there exists a template to safely introduce Binary tags in signals:

```text
template binaryCheck () {
   signal input in;
   signal:Binary output out;
   in * (in-1) === 0;
   out <== in;
}
```

Now, using these library templates, a developer can build new circuits combining safely existing library subcomponents. The following example shows this.

```text
template Main () {
   signal input in1;
   signal input in2;
   signal input in3;
   signal output out;

   component a1 = binaryCheck();
   component a2 = binaryCheck();
   component a3 = binaryCheck();

   a1.in <== in1; 
   a2.in <== in2; 
   a3.in <== in2; 
   component and1 = AND();
   component and2 = AND();

   and1.in1 <== a1.out;
   and1.in2 <== a2.out;
   and2.in1 <== and1.out;
   and2.in2 <== a3.out;
   out <== and2.out;
}

component main {public [in1,in2]}= Main();
```

