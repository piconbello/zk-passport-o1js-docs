# LDS → SignedAttrs

This step needs to do/check the following:

1. `DG1` SHA-digest is indeed inside the correct place in `LDS`
2. `LDS` is SHA-digested and is propogated to the next step.

Since `LDS` can be large enough to SHA-digest in a single circuit, we are making use of our [multi-circuit digest](../utilities/digest.md) to SHA-digest it.

We prepare:

1. 0 or more update circuits with full blocks
2. 1 finalize circuit with a dynamically-sized block
3. The actual `LDS` validation circuit described below.

The `carry` of the digest is the Poseidon-digest of SHA-digest of `DG1`, which is checked against previous step's `out.right`, by Poseidon-digesting the input SHA-digest or `DG1`.

### Implicit Variance

There needs to be a separate circuit for each SHA variant.

For example, we have a circuit for `sha256` variant for LDS.

### Private Inputs

| Input            | Type           |
| ---------------- | -------------- |
| `DG1 SHA-digest` | `Bytes`        |
| `DigestState`    | `DigestState`  |
| `LDS`            | `DynamicBytes` |

### Public Output

| Left                                      | Right                                  |
| ----------------------------------------- | -------------------------------------- |
| Poseidon-digest of the last `DigestState` | Poseidon-digest of SHA-digest of `LDS` |
