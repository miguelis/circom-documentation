---
description: This tutorial shows how to build a circuit  implementing an elliptic curve.
---

# The baby jubjub elliptic curve

Finally, we are going to use circom to define the baby jubjub elliptic curve. The technical details of this curve can be found [here](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/research/publications/zkproof-standards-workshop-2/baby-jubjub/baby-jubjub.html) and [here](https://iden3-docs.readthedocs.io/en/latest/_downloads/33717d75ab84e11313cc0d8a090b636f/Baby-Jubjub.pdf). The circom code can be found [here](https://github.com/iden3/circomlib/blob/master/circuits/babyjub.circom). We have decided to use this curve to illustrate the potential of the language since it is used in the real cryptographic world, but it is also quite simple to understand by non-experts in elliptic curves.

## Checking if a point belongs to the curve

This curve is defined by two coefficients, a and d, whose values are 168700 and 168696, respectively. These coefficients are going to be used in all the templates related to this curve. We do not want to replicate the definition of these values in every template, since this practice is prone to error. However, circom does not admit the definition of global constants, then the best solution is to define two functions that return always these values.

```text
function Baby_const_a(){                         
	 return 168700;                                    
}                                                

function Baby_const_d(){
	 return 168696;
}
```

This way we only have to call these functions instead of writing each value in every template in which we need them.

The first template that we are going to define will check if a given point belongs to the curve. Every point of this curve satisfies the next equation: $$ax^2+y² = 1 + dx²y²$$ . 

```text
template BabyCheck() {
    signal input x;
    signal input y;
    var a = Baby_const_a();
    var d = Baby_const_d();
    signal x2;
    signal y2;
    x2 <== x*x;
    y2 <== y*y;
    a*x2 + y2 === 1 + d*x2*y2;
}
```

First, we declare two input signals: one for each coordinate of the point to be checked. After that, we obtain the values of coefficients `a` and `d` from the functions previously defined. It is important to highlight that we cannot directly write the constraint `ax²+y²=== 1 + dx²y²` since it is not a quadratic constraint.  We need two new intermediate signals, `x2` and `y2`, to express `x²` and `y²`. Once we have these signals defined, we can check if the point belongs to the curve using a quadratic constraint:`ax2+y2 === 1 + d*x2*y2`.

## Addition of points in the curve

Let us define how to operate in the elliptic curve group: we need to define a template for the addition of points and, later, a template for multiplication of a point by a scalar \(an element of $$F_p$$\). 

Given two points$$p_1 = (x_1,y_1)$$ and $$p_2 = (x_2,y_2)$$, their addition is defined as a point$$p_3=(x_3,y_3)$$, where $$x_3 = \frac{x_1y_2+x_2y_1}{1+dx_1x_2y_1y_2}$$  and $$y_3 = \frac{y_1y_2 - ax_1x_2}{1-dx_1x_2y_1y_2}$$ .

```text
template BabyAdd() {
    signal input p1[2];
    signal input p2[2];
    signal output pout[2];

    signal beta;
    signal gamma;
    signal delta;
    signal tau;
    var a = Baby_const_a();
    var d = Baby_const_d();
    beta <== p1[0]*p2[1];
    gamma <== p1[1]*p2[0];
    delta <== (-a*p1[0]+p1[1])*(p2[0] + p2[1]);
    tau <== beta * gamma;

    pout[0] <-- (beta + gamma) / (1+ d*tau);
    (1+ d*tau) * pout[0] === (beta + gamma);

    pout[1] <-- (delta + a*beta - gamma) / (1-d*tau);
    (1-d*tau)*pout[1] === (delta + a*beta - gamma);
}

```

In this example, we define a point using a 2-dimension array of signals. Then, we define two points as input signals, the third point as output signal, four intermediate signals, and the two coefficients of the curve. It is important to highlight that the value of the output signals cannot be expressed like a quadratic constraint due to the division. Consequently, we cannot use operator `<==`, and we have to use `<--` to set the value of signals `pout[0]` and `pout[1]`, and `===` to define the constraints among them and the input signals.

The use of the operator `<==` guarantees an equivalence between the values of the signals after executing the code of the program and the constraints produced during the compilation. The use of the operators `<--` and `===` does not guarantee this equivalence and, thus, they must only be used  in cases in which the operator `<==` cannot be used, just like in the previous example. [Here](../intro/common-programming-concepts/signals/) you can find more details about these operators.

Due to the loss of equivalence between code and constraints, we can find cases in which the expected witness does not produce erroneous constraints and, other cases, in which the constraints are satisfied by unexpected witnesses.  

## **Multiplication of a point and a scalar in the curve**

The most important operation in the curve is the multiplication of a point in the curve and a scalar, obtaining a new point which also belongs to the curve. We say that the first point is a generator point.

For the security of the curve, it is very important to choose a good generator point. If it is not good enough, the curve will be easily attacked. The property that this point must satisfy is defined [here](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/research/publications/zkproof-standards-workshop-2/baby-jubjub/baby-jubjub.html#pdf-link). It basically claims that a point is a good generator if we can obtain a huge number of different points from it. 

It is relatively easy to calculate a new point from the generator and a scalar in the field. However, if we know the generator and a point in the curve, it is quite difficult to calculate the scalar used to generate it. Consequently,  the key of this theory is that we can use a point in the curve as a public key for a cryptographic protocol, whereas the scalar used to generate such a point remains as a private key.

```text
template BabyPbk() {
    signal input in;
    var BASE8[2] = [
        5299619240641551281634865583518297030282874472190772894086521144482721001553,
        16950150798460657717958625567821834550301663161624707787222815936182638968203
    ];
    signal output Ax;
    signal output Ay;
    component pvkBits = Num2Bits(253);
    pvkBits.in <== in;
    component mulFix = EscalarMulFix(253, BASE8);
    var i;
    for (i=0; i<253; i++) {
        mulFix.e[i] <== pvkBits.out[i];
    }
    Ax  <== mulFix.out[0];
    Ay  <== mulFix.out[1];
}
```

Let us see the main parts of this template. We have an input signal `in`, which is the scalar used to generate the new point, the generator point `BASE8[2]` \(you can judge the size of the point by scrolling the window\), and two output signals, `Ax` and `Ay`, which are the coordinates of the point generated from multiplying `BASE8[2]` and `in`. After these definitions, we declare a component to transform the scalar in its 253-bit representation \([Num2Bits](https://github.com/iden3/circomlib/blob/master/circuits/bitify.circom)\) and we assign the scalar to its input signal. Now, we declare a new component to perform the multiplication of the generator and the scalar \([EscalarMulFix](https://github.com/iden3/circomlib/blob/master/circuits/escalarmulfix.circom)\). Details about how this multiplication is performed can be found [here](https://iden3-docs.readthedocs.io/en/latest/_downloads/33717d75ab84e11313cc0d8a090b636f/Baby-Jubjub.pdf). Each of the bits from the representation of the scalar are set to its corresponding input of the component. Finally, we assign the output signals of this component to the final output signals `Ax` and `Ay`.

```text
component main = BabyPbk()
```

We can compile the program without declaring any input signal as public, then they remain as private signals. In particular, `in` is a private signal whose value cannot be known for the security of the curve.

