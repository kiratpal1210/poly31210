# zardkat 🐱

A [hardhat-circom](https://github.com/projectsophon/hardhat-circom) template to generate zero-knowledge circuits, proofs, and solidity verifiers

## Quick Start
Compile the kiratCircuit() circuit and verify it against a smart contract verifier

```
pragma circom 2.0.0;


template kiratCircuit() {  
  
  //signal inputs

  signal input a;
  signal input b;

  //signals from gates

  signal x;
  signal y;

  //final signal output

  signal output q;

  //component gates used to create custom circuit

  component andGate = aND();
  component notGate = NOT();
  component orGate = OR();

  //circuit logic 

  andGate.a <== a;
  andGate.b <== b;
  x <== andGate.out;

  notGate.in <== b;
  y <== notGate.out;

  orGate.a <== x;
  orGate.b <== y;
  q <== orGate.out;
}

template aND() {
    signal input a;
    signal input b;
    signal output out;

    out <== a * b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2 * in;
}

template OR() {
    signal input a;
    signal input b;
    signal output out;

    out <== a + b - a * b;
}

component main = kiratCircuit();
```
### Install
`npm i`

### Compile
`npx hardhat circom` 
This will generate the **out** file with circuit intermediaries and geneate the **contracts/kiratCircuitVerifier.sol** contract

### Prove and Deploy
`npx hardhat run scripts/deploy.ts`
This script does 4 things  
1. Deploys the contracts/kiratCircuitVerifier.sol contract
2. Generates a proof from circuit intermediaries with `generateProof()`
3. Generates calldata with `generateCallData()`
4. Calls `verifyProof()` on the verifier contract with calldata

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── kiratCircuit
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── kiratCircuit.r1cs
│       ├── kiratCircuit.vkey
│       └── kiratCircuit.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── kiratCircuitVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **kiratCircuit**

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "kiratCircuit",
    "protocol": "groth16",
    "circuit": "kiratCircuit/circuit.circom",
    "input": "kiratCircuit/input.json",
    "wasm": "kiratCircuit/out/circuit.wasm",
    "zkey": "kiratCircuit/out/kiratCircuit.zkey",
    "vkey": "kiratCircuit/out/kiratCircuit.vkey",
    "r1cs": "kiratCircuit/out/kiratCircuit.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

**adding circuits**   
To add a new circuit, you can run the `newcircuit` hardhat task to autogenerate configuration and directories i.e  
```
npx hardhat newcircuit --name newcircuit
```

**determinism**
> When you recompile the same circuit using the groth16 protocol, even with no changes, this plugin will apply a new final beacon, changing all the zkey output files. This also causes your Verifier contracts to be updated.
> For development builds of groth16 circuits, we provide the --deterministic flag in order to use a NON-RANDOM and UNSECURE hardcoded entropy (0x000000 by default) which will allow you to more easily inspect and catch changes in your circuits. You can adjust this default beacon by setting the beacon property on a circuit's config in your hardhat.config.js file.

**Authors**
Kiratpal Singh kalsey
https://www.linkedin.com/in/kiratpal-singh-kalsey-a92b15230/
