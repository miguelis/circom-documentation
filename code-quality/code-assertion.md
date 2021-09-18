# Code assertion

**assert\(bool\_expression\);**

This statement introduces conditions to be checked at construction time only. If the condition fails, the compilation is aborted and the error is reported. It does not generate any constraint.

```text
template Translate(n) {
assert(n<=254);
…..
}
```

The assert expression cannot depend on unknowns and will be checked at construction time. The next template will produce a compilation error.

```text
template wrong(){
    signal input in;
    assert(0 < in);
}
```

Recall that, instead, when a constraint is introduced with ===, then an assert is automatically added in the witness generation code.

