# Field elements

A field element is a value in the domain of Z/pZ, where p is the prime number set by default to

`p = 21888242871839275222246405745257275088548364400416034343698204186575808495617.`

As such, field elements are operated in arithmetic modulo p.

The Circom language is parametric to this number, and it can be changed without affecting the rest of the language \(using `GLOBAL_FIELD_P`\).

