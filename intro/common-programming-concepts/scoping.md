# Scoping

Circom has a static scoping like in C or Rust. However, we have that signals and components must have a global scoping and hence they should be defined at the top-level block of the template that defines them. Regarding visibility, a signal x of component c is also visible in the template t that has declared c, using the notation c.x as described above. No access to signals of nested sub-components is allowed. For instance, if c is build using another component d, the signals of d cannot be accessed from t.

A var can be defined at any block and its visibility is reduced to the block like in C or Rust.

