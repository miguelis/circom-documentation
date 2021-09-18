# Signal Tags

In order to help those programmers of Circom that mainly combine library components, signals can be decorated with tags that are checked at compilation time. In Circom 1.0, there are only two predefined tags:

* FieldElement, which means that a general numeric value of the field \(modulo p\) is expected. This is the default.
* Binary, which means that only 0 and 1 are expected as values.

```text
signal:Binary input in;
signal:Binary output out[n];
```

Currently, no user-defined tags can be introduced, and they have very limited use in Circom:

_Signal tags do not introduce any constraints or change the type of the signal. However, it imposes a syntactic condition that is checked by the compiler. In particular, if an input signal is tagged to be Binary then it can only be assigned a value with ==&gt; or &lt;== using a Binary tagged signal. Currently, the FieldElement tag implies no check as all signals are intended to be elements from the field._

[Safe design of circuits using templates with signal tags](../template/safe-design-of-circuits-using-templates-with-signal-tags.md) describes how the Binary tag is used in library templates to ensure that, if no Binary tag is directly introduced by the user \(without using a library component\) then a circuit built only using library components is safe in the sense that it has all desired constraints.

