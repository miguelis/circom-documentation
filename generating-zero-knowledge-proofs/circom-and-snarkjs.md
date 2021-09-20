---
description: >-
  circom allows the creation of circuits for zk-proofs. The output of circom is
  compatible with snarkJS, a JavaScript implementation for zk-SNARKs. We explain
  step-by-step how to combine the two tools.
---

# circom & snarkJS

## Step-by-step <a id="step-by-step"></a>

* Write your arithmetic circuit using circom and save the file with `.circom` extension. Remember that you can create your own circuits or use the templates from the library of circuits [`circomlib`](https://github.com/iden3/circomlib).
* Compile the circuit to get a system of arithmetic equations representing the circuit.

```text
$ circom circuit.circom --r1cs --wasm --sym
```

* Now, use `snarkJS` to generate and verify zk-SNARK proofs.
* First, introduce the inputs to your circuit and compute the corresponding [witness](https://docs.circom.io/1.-an-introduction/background#witness).

```text
$ snarkjs wtns calculate circuit.wasm input.json witness.wtns
```

* Before generating the proof, you will need to generate a [trusted setup](https://docs.circom.io/1.-an-introduction/background#trusted-setup). Generating a trusted setup consists of 2 parts: powers of tau and phase 2. We will not go deep into this topic here, we recommend you to review our [Getting started](https://docs.circom.io/1.-an-introduction/getting-started) section or to read directly [the snarkJS tutorial](https://github.com/iden3/snarkjs).

```text
// Start a new powers of tau ceremony and make a contribution (enter some random text)​$ snarkjs powersoftau new bn128 12 pot12_0000.ptau -v$ snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v​// Prapare phase 2​$ snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v​// Start a new zkey and make a contribution (enter some random text)​$ snarkjs zkey new circuit.r1cs pot12_final.ptau circuit_0000.zkey$ snarkjs zkey contribute circuit_0000.zkey circuit_final.zkey --name="1st Contributor Name" -v
```

* Export the verification key from `circuit_final.zkey`.

```text
$ snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
```

* Generate a zero-knowledge proof associated to the circuit and your witness.

```text
$ snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
```

* Once you have the proof, it is possible to validate the proof by running the following command.

```text
$ snarkjs groth16 verify verification_key.json public.json proof.json
```

It is also possible to generate a Solidity verifier that allows verifying proofs on Ethereum blockchain. To this end, you should:

* Generate the Solidity code by typing the following.

```text
$ snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol
```

* Cut and paste in [Remix](https://remix-project.org/) the code from the output file named `verifier.sol` . You only need to deploy the `Verifier` contract.

### [Would you like to know more about `snarkJS`? Start using it now!](https://docs.circom.io/1.-an-introduction/getting-started) <a id="would-you-like-to-know-more-about-snarkjs-start-using-it-now"></a>

## Visual summary <a id="visual-summary"></a>

![](https://gblobscdn.gitbook.com/assets%2F-MDt-cjMfCLyy351MraT%2F-ME35kSLplV3Z39JJsLE%2F-ME37Q2MlDc67k0-jzQS%2Fcircomsnarkjs.png?alt=media&token=4b1b1c11-a1d4-4048-8c3a-0c7b02f4930a)

