# Templates & Components

The mechanism to create generic circuits in Circom is the so-called templates.

They are normally parametric on some values that must be instantiated when the template is used. The instantiation of a template is a new circuit object, which can be used to compose other circuits, so as part of larger circuits. Since templates define circuits by instantiation, they have their own signals \(input, output, etc\).

```text
template tempid ( param_1, ... , param_n ) {
 signal input a;
 signal output b;

 .....

}
```

In this version, templates cannot include local functions or template definitions.

Assigning a value to an input signal inside the same template where it has been defined also generates an error.

The instantiation of a template is made using the keyword component and by providing the necessary parameters.

```text
component c = tempid(v1,...,vn);
```

The values of the parameters should be known constants at compile time.

Regarding the signals defined in the template that will be part of the component, the following compiler messages will be generated:

1. If a signal is not used in any constraint, an error message will be generated. Moreover, if it is an input signal x then the compiler would suggest adding a constraint of the form x \* 0 === 0;
2. If an intermediary signal is used only in one constraint, a hint message will be generated.
3. If there is no output signal a warning message will be generated. 

The compiler would suggest adding a validation Binary output \(that checks the inputs\).

The component instantiation will not be triggered until all its input signals are assigned to concrete values. Therefore the instantiation might be delayed and hence the component creation instruction does not imply the execution of the component object, but the creation of the instantiation process that will be completed when all the inputs will be set. _The output signals of a component can only be used when all inputs are set, otherwise a compiler error is generated_. For instance, the following piece of code would result in an error:

```text
template Internal() {
   signal input in[2];
   signal output out;
   out <== in[0]*in[1];
}

template Main() {
   signal input in[2];
   signal output out;
   component c = Internal ();
   c.in[0] <== in[0];
   c.out ==> out;  // c.in[1] is not assigned yet
   c.in[1] <== in[1];  // this line should be placed before calling c.out
}

component main = Main();
```

A component defines an arithmetic circuit and, as such, it receives N input signals and produces M output signals and K intermediate signals. Additionally, it can produce a set of constraints.

Components are immutable \(like signals\). A component can be declared first and initialized in a second step. If there are several initialization instructions \(in different execution paths\) they all need to be instantiations of the same template \(maybe with different values for the parameters\).

In order to access the input or output signals of a component, we will use dot notation. No other signals are visible outside the component.

```text
c.a <== y*z-1;
var x;
x = c.b;
```

We can define arrays of components following the same restrictions on the size given before. Moreover, initialization in the definition in arrays of components is not allowed, and instantiation can only be made component by component, accessing the positions of the array. All components in the array have to be instances of the same template.

```text
template MultiAND(n) {
...
component ands[2];
ands[0] = MultiAND(n1);
ands[1] = MultiAND(n2);
...
}
```



