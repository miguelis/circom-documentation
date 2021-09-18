---
description: This tutorial will guide you through the installation of circom and snarkJS.
---

# Installation

First of all, you need to install Rustup. If you’re using Linux or macOS, open a terminal and enter the following command:

```text
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

If the installation is successful, you will receive the message:

```text
Rust is installed now. Great!
```

Once Rustup is installed, you have to checkout the github repository of the circom compiler. 

```text
git clone https://github.com/hermeGarcia/circom_compiler.git . 
```

If you are used to subversion, you can use: 

```text
svn checkout https://github.com/hermeGarcia/circom_compiler.git/trunk . 
```

Both commands will install the repository at the current directory.

{% hint style="danger" %}
If you have not installed gcc, you must install it to proceed to the final step. You can simply do it using the next command:

`sudo apt install build-essential`
{% endhint %}

Finally, you use the command cargo to build the compiler in the main directory of circom compiler.

```text
cargo build --release
```

The installation takes around 3 minutes to be completed.  If the installation is successful, you will receive the next message:

```text
   Finished release [optimized] target(s) in xx m xx s
```

This command will generate the `circom` executable  in the directory `circom/target/release`. You can see all the options of the executable by using the flag `help`:

```text
circom --help
```

## Installing snarks-js <a id="installing-the-tools"></a>

First, you need to install `Node.js`. To get a better performance, it is important that you install version 10 or higher. This is because the last versions of `Node.js` include big integer support and web assembly compilers that help run code faster.

Then, install `snarkJS`. You can do it by running the following command:

```text
npm install -g snarkjs
```

