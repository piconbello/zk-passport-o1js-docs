# DG1 &rarr; LDS

We need to first SHA-digest DG1, in order to prove that it exists in the expected position in LDS in later steps.

This step solely digests the DG1.

## Implicit Variance

There needs to be a separate circuit for each (`DG1` variant, SHA variant) pair.

For example, we have a circuit for `TD3,sha256` pair.

## Private Inputs

| Input | Type    |
| ----- | ------- |
| `DG1` | `Bytes` |

## Public Output

`carry` is Poseidon-digest of SHA-digest of `DG1`.

`right` is `DigestState.init` with `carry` included.

| Left                                   | Right |
| -------------------------------------- | ----- |
| Poseidon-digest of `DigestState+carry` |       |
