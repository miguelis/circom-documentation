---
description: >-
  This tutorial will guide you through the process of creating and validating
  your first zero-knowledge SNARK circuit using circom and snarkJS.
---

# Our first zero-knowledge proof

After the previous sections, we are now ready to **compile** a `circom` circuit.

Let us consider the `Multiplier2` circuit from [Our first circom program](our-first-circom-program.md). 

```text
template Multiplier2() {
    signal private input a;
    signal private input b;
    signal output c;
    c <== a*b;
 }

 component main = Multiplier2();   
```

 Run the following command:

```text
circom multiplier2.circom --r1cs --wasm --sym
```

With these options we generate three types of files:

* `--r1cs`: it generates the file `circuit.r1cs` that contains the [R1CS constraint system](../background.md#rank-1-constraint-system) of the circuit in binary format.
* `--wasm`: it generates the file `circuit.wasm` that contains the `Wasm` code to generate the [witness](../background.md#witness).
* `--sym` : it generates the file `circuit.sym` , a symbols file required for debugging or for printing the constraint system in an annotated mode.

## My first zero-knowledge proof <a id="my-first-zero-knowledge-proof"></a>

Now that the circuit is compiled, we will use `snarkJS` to generate and validate zk-SNARK proofs. More specifically, **we will prove that we are able to factor the number 33**. That is, we will show that we know two integers `a` and `b` such that when we multiply them, it results in 33.

You can always access the **help** of`snarkJS` by typing the command

`$ snarkjs --help`

1. With `snarkJS`, you can get general **statistics** of the circuit and print the **constraints**. Just run:

```text
snarkjs info -c circuit.r1cssnarkjs print -r circuit.r1cs -s circuit.sym
```

 2. To generate and verify zk-SNARK proofs, it is required to have a **setup**.

The generation of the setup has two phases: the first step consists in the creation of some values known as **powers of tau**. For this tutorial we will keep a simple "powers of tau" file. Run the following commands:

```text
// Start a new powers of tau ceremony and make a contribution​snarkjs powersoftau new bn128 12 pot12_0000.ptau -vsnarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

You will be prompted to **enter some random text** that will be used as source of entropy. Once you have entered you text, the commands will output a transcript named `pot12_001.ptau`. This first part of the setup is generic and useful to any circuit, so you can save it to reuse it in future projects.

The generation of the **setup** is necessary to obtain the proving and verification keys of zk-SNARK proofs. This step requires the generation of some random values that need to be eliminated. This elimination process is crucial: if these values are ever exposed, the security of the whole scheme is compromised.

To construct the setting, we use a [multi-party computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation) \(MPC\) ceremony that allows multiple independent parties to collaboratively construct the parameters. With MPC, it is enough that **one single participant deletes its secret** counterpart of the contribution in order to keep the whole scheme secure.

The construction of the trusted setup has two phases: a general MPC ceremony that is valid for any circuit \(known as **powers of tau ceremony**\), and a second phase \(**phase 2**\) that is constructed for each specific circuit. Anyone can contribute with their randomness to the MPC ceremonies and typically, before getting the final parameters, a [random beacon](https://eprint.iacr.org/2017/1050) is applied.

For the sake of simplicity, we generated a setup with only one contribution without giving all the details. If you want to deepen into robust setups with multiple contributions:

* You can [download](https://www.dropbox.com/sh/mn47gnepqu88mzl/AACaJkBU7mmCq8uU8ml0-0fma?dl=0) the file `powersOfTau28_hez_final.ptau`, which contains 54 contributions and a final random beacon, and use it to import and export challenges.
* If you want to run your own MPC ceremonies or learn about generating trusted setups with `snarkJS`, we recommend you to follow [this other tutorial](https://github.com/iden3/snarkjs).

The next step, which is called **phase 2** of the setup, is **circuit-specific**. First, we have to prepare the phase running the command:

```text
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v
```

The generation of phase 2 is similar to what we did with powers of tau. In this case, we will generate a `.zkey` file that will contain the proving and verification keys together with all phase 2 contributions.

```text
// Start a new zkey and make a contribution​snarkjs zkey new circuit.r1cs pot12_final.ptau circuit_0000.zkeysnarkjs zkey contribute circuit_0000.zkey circuit_final.zkey --name="1st Contributor Name" -v
```

As before, you will be prompted to enter some random text to provide a source of entropy. The output will be a file named `circuit_final.zkey`, which we will use to **export the verification key**.

```text
snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
```

Now, the verification key from `circuit_final.zkey` is exported into the file `verification_key.json`.

You can always **verify** that the computations of a `.ptau` or a `.zkey` file are correct:

```text
snarkjs powersoftau verify pot12_final.ptausnarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final.zkey
```

If everything checks out, you should see the following at the top of the output:

```text
[INFO]  snarkJS: Powers of Tau file OK![INFO]  snarkJS: ZKey OK!
```

​The command `snarkjs zkey verify` also checks that the `.zkey` file corresponds to the specific circuit.


 3. Once the setup is ready, we **compute a** **witness** for the circuit.

Before creating any proof, we need to calculate all the signals of the circuit that match all the constraints of the circuit. For that, we will use the `Wasm` module generated by`circom` that helps do this job. We simply need to provide a file with the inputs and the module will execute the circuit and calculate all the intermediate signals and the output. The set of inputs, intermediate signals and output is called [witness](../background.md#witness).

In our case, we want to prove that we able to factor the number 33. So, we assign `a = 3` and `b = 11`.



Note that we could assign the number 1 to one of the inputs and the number 33 to the other. So, our proof does not really show that we are able to factor the number 33. [At the end of this section](our-first-zero-knowledge-proof-need-more-changes-with-the-example.md#my-first-zero-knowledge-proof), we will add few modifications to the circuit to deal with this problem.

We need to create a file named `input.json` with the inputs.

```text
{"a": 3, "b": 11}
```

Now, we calculate the witness and generate a file `witness.wtns` with the result.

```text
snarkjs wtns calculate circuit.wasm input.json witness.wtns
```

 4. Once the witness is computed, we can **generate a zk-proof** associated to the circuit and the witness.

```text
snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
```

This command generates a proof \([Groth16](https://eprint.iacr.org/2016/260) is a specific zk-SNARK protocol\) and outputs two files:

* `proof.json`: it contains the zk-SNARK proof.
* `public.json`: it contains the values of the public inputs and outputs.

 5. To **verify the proof**, run:

```text
snarkjs groth16 verify verification_key.json public.json proof.json
```

This command uses the files `verification_key.json` we exported earlier,`proof.json` and `public.json` to check if the proof is valid. If the proof is valid, the command outputs an `OK`.

A valid proof not only proves that we know a set of signals that satisfy the circuit, but also that the public inputs and outputs that we use match the ones described in the `public.json` file.

​👉 It is also possible to generate a **Solidity verifier** that allows **verifying proofs on Ethereum blockchain**.

First, we need to generate the Solidity code using the command:

```text
snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol
```

This command takes validation key `circuit_final.zkey` and outputs Solidity code in a file named `verifier.sol`. You can take the code from this file and cut and paste it in Remix. You will see that the code contains two contracts: `Pairing` and `Verifier`. You only need to deploy the `Verifier` contract.

You may want to use first a testnet like Rinkeby, Kovan or Ropsten. You can also use the JavaScript VM, but in some browsers the verification takes long and the page may freeze.

The `Verifier` has a `view` function called `verifyProof` that returns `TRUE` if and only if the proof and the inputs are valid. To facilitate the call, you can use `snarkJS` to generate the parameters of the call by typing:

```text
snarkjs generatecall
```

Cut and paste the output of the command to the parameters field of the `verifyProof` method in Remix. If everything works fine, this method should return `TRUE`. You can try to change just a single bit of the parameters, and you will see that the result is verifiable `FALSE`.

