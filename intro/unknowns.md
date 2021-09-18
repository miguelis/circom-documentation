# Unknowns

The concept of unknown is very important in the constructive part of Circom, where constraints are generated. In order to understand the compiler behavior we need to define what is considered unknown at compile time.

As already said, the content of a signal is always considered unknown, and only constant values or template parameters are considered known. A var whose value depends on unknowns is unknown. Similarly, any expression that depends on unknowns is considered unknown. Additionally, if an array is modified with an unknown expression in an unknown position then all positions of the array become unknown. Finally, the result of a function call with unknown parameters is unknown.

The key point for the compiler in the constructive phase is that the generation of constraints cannot depend on conditions \(expressions\) that are unknown. In the next section, this is imposed on all statements.

