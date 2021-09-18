# Code assertion

**assert\(bool\_expression\);**

This statement introduces conditions to be checked both at construction time only. The assert expression cannot depend on unknowns and will be checked at construction time. If the condition fails, the compilation is aborted and the error is reported. It does not generate any constraint.

```text
template Translate(n) {
assert(n<=254);
…..
}
```

Recall that, instead, when a constraint is introduced with ===, then an assert is automatically added in the witness generation code.

