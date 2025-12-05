# leo-zk-sonic-rollup

Minimal Leo program that models a confidential transfer step, inspired by web3 zk-rollup and privacy projects such as Aztec, Zama, and soundness-focused proof systems.

The circuit enforces that a transfer between two confidential notes:
- Matches publicly known commitments.
- Conserves value (no hidden inflation).
- Applies a public fee parameter.

Repository contains exactly two files:

1. app.zk_sonic.leo – Leo program implementing the zk soundness circuit.
2. README.md – this documentation file.

---

## Concept

High-level idea:

- Each balance is represented as a confidential note:
  - value: the amount as a u64.
  - blinding: a field element providing randomness.
- A Pedersen-style commitment binds (value, blinding) into a public field element.
- The Leo program takes:
  - Public inputs:
    - old_commitment
    - new_commitment
    - fee
  - Private inputs:
    - old_value, old_blinding
    - new_value, new_blinding
- It enforces the following:
  - Commitments are sound:
    - The old note and new note, when recomputed from private values, match the provided public commitments.
  - Value conservation:
    - old_value = new_value + fee.
  - Basic sanity constraints:
    - old_value > 0, new_value > 0, fee <= old_value.

This mirrors core building blocks used by:

- Aztec-style rollups, which maintain private notes and public commitments while guaranteeing soundness with proofs.
- Zama-style FHE systems, where encrypted computation outputs can be checked with zk circuits.
- General soundness frameworks for web3, L2s, and rollups that require provable conservation of value.

---

## Repository structure

- app.zk_sonic.leo  
  Leo program containing:
  - Note struct for confidential balances.
  - note_commitment function using a Pedersen-style hash.
  - main transition that enforces soundness and value conservation.

- README.md  
  This documentation file with installation, usage, and design notes.

---

## Prerequisites

To build and run this repository you should have:

- Leo language toolchain installed (Aleo ecosystem).
- A Rust toolchain if required by your Leo installation.
- Basic understanding of:
  - Zero-knowledge proofs and circuits.
  - Commitments and blinding factors.
  - How Aztec-like rollups or Zama-style systems use proofs for soundness.

Assumptions:

- You can create and manage Leo projects from the command line.
- You are familiar with the basic `leo new`, `leo build`, and `leo run` flows (or equivalent in your Leo version).

---

## Installation

1. Create the project directory

Choose a folder name, for example:
- leo-zk-sonic-rollup

2. Initialize a Leo project

Use the standard Leo command to create a new project in this folder. This should create a project structure with a `src` directory and a configuration file.

3. Add the program file

Inside the `src` directory:
- Create a file named `app.zk_sonic.leo`.
- Copy the contents of `app.zk_sonic.leo` from this repository into that file.

4. Wire the file into the project

Depending on your Leo version, you have two common options:

- Option A: Make `app.zk_sonic.leo` the main program file:
  - Either rename `app.zk_sonic.leo` to the default main file name expected by Leo, or
  - Update your configuration to use `app.zk_sonic.leo` as the primary source file.

- Option B: Import the program from another file:
  - Keep your existing main file.
  - Import the `zk_sonic.aleo` program defined in `app.zk_sonic.leo` and call its `main` transition from your top-level code.

5. Ensure standard library access

The program assumes access to a Pedersen-style hash function via the Leo standard library:
- If your Leo version uses a different import path or name for Pedersen hashing, adjust the `std::hash::pedersen` usage inside `app.zk_sonic.leo` while preserving the conceptual structure.

---

## Building

To build the project:

1. Navigate to the project root (containing the main Leo configuration file).
2. Run the build command for your Leo version.
3. Verify that:
   - The compiler detects `app.zk_sonic.leo`.
   - The program `zk_sonic.aleo` with its `main` transition compiles successfully.

If you encounter compilation errors:

- Check that the file extension and program name match your Leo toolchain expectations.
- Confirm that the `std::hash::pedersen` usage is compatible with your version of the standard library.
- Ensure there are no conflicting program names in other files.

---

## Running and proving

Once the project builds, you can run and prove the circuit with Leo commands.

1. Prepare inputs

You will need to supply:

- Public inputs:
  - old_commitment (field)
  - new_commitment (field)
  - fee (u64)

- Private inputs:
  - old_value (u64)
  - old_blinding (field)
  - new_value (u64)
  - new_blinding (field)

These may be provided through input files or CLI prompts depending on your Leo version.

2. Execute the transition

Run the `main` transition of the `zk_sonic.aleo` program with your chosen inputs.

On success:

- The program returns `(old_commitment, new_commitment, fee)` as public outputs.
- The proof system guarantees that:
  - Commitments were computed from the private values as claimed.
  - Value conservation and sanity constraints hold.

3. Verify the proof

Use the standard verification command for your Leo environment:

- Provide the generated proof and the public inputs.
- Verification success means that no violation of the encoded constraints is possible given the supplied proof.

---

## Integration with web3 and rollups

This program is designed as a small building block for web3 and L2 systems:

- Aztec-style rollups
  - Store commitments on chain.
  - Maintain private note data off chain.
  - Use circuits like this to ensure that updates to commitments are sound.

- Zama-style FHE integrations
  - Perform encrypted computations on balances off chain.
  - Use Leo circuits to prove that decrypted transitions match the encrypted operations.

- Soundness-focused zk frameworks
  - Rely on invariants like value conservation and commitment consistency.
  - Extend circuits like this to include:
    - Multiple input and output notes.
    - Nullifier checks to prevent double spending.
    - Merkle tree membership proofs.

Typical workflow:

- Maintain a database of notes and commitments.
- When a user wants to transfer or withdraw:
  - Construct the witness values (old_value, old_blinding, new_value, new_blinding, fee).
  - Compute commitments and run the Leo circuit.
  - Generate a proof and submit it to a verifier (on chain or off chain).
- A successful verification ensures that:
  - No money was created out of thin air.
  - Commitments and witnesses are consistent.

---

## Expected result

After integrating this repository into a Leo project and running the circuit, you should be able to:

- Compile the program without errors.
- Provide test inputs and generate a proof that:
  - Confirms the soundness of a single confidential transfer step.
  - Ensures that `old_value = new_value + fee`.
- Observe that:
  - Any mismatch in commitments causes the proof to fail.
  - Any attempt to violate value conservation or sanity constraints also fails.

Conceptually, the expected outcome is a compact, understandable example of:

- How Leo can encode soundness constraints for web3 rollups.
- How commitments, notes, and fees interact in a zk circuit.
- How ideas from Aztec, Zama, and broader soundness-focused ecosystems can be prototyped with Leo.

---

## Notes and limitations

- Simplified model:
  - Only one input note and one output note plus a fee.
  - No nullifiers, Merkle trees, or note sets.
- Privacy and metadata:
  - Public values are limited to commitments and fee.
  - Additional privacy guarantees may require more constraints and careful protocol design.
- Security and production readiness:
  - This repository is educational and not audited.
  - It omits:
    - On-chain verifier contracts.
    - Wallet integration.
    - Complex protocol features like multi-asset support or batching.
- Version compatibility:
  - Leo and its standard library evolve over time.
  - You may need to adapt imports, function names, or syntax to match your specific toolchain.

---

## Future extensions

To evolve this repository into a richer protocol component, consider:

- Supporting multiple input and output notes in `main`.
- Adding nullifier logic to prevent double spending.
- Including Merkle tree membership proofs for notes.
- Integrating with a smart contract or L1 verifier that checks Leo proofs on chain.
- Connecting to FHE-based systems like Zama for encrypted computation.
- Building a front-end or CLI layer that:
  - Creates notes and commitments.
  - Tracks balances and fees.
  - Automates proof generation and verification.

This repository is intended as a clean and concise starting point for experimenting with Leo, Aleo, and web3 soundness patterns inspired by Aztec, Zama, and general zk ecosystems.

