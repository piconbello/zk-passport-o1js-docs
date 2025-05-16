# SHA-digesting SignedAttrs

We prove that SHA-digest of `LDS` is at the correct place in the `SignedAttrs` payload.

We then digest the `SignedAttrs` for the signature verification for the next step.

### Implicit Variance

The digest algorithm of the `DG1 → LDS` and `LDS → SignedAttrs` must be the same SHA size. However, the signing of the `SignedAttrs` may use a different SHA size.

Therefore there needs to be a separate circuit for each pair of SHA variants.

For example, we have a circuit for `sha256,sha256` variant for `SignedAttrs`.

First SHA size determines the size of `SignedAttrs`, also the offset of the SHA-digest of `LDS` in it.

### Private Inputs

| Input         | Type    |
| ------------- | ------- |
| `LDS digest`  | `Bytes` |
| `SignedAttrs` | `Bytes` |

### Public Output

| Left                                   | Right                                          |
| -------------------------------------- | ---------------------------------------------- |
| Poseidon-digest of SHA-digest of `LDS` | Poseidon-digest of SHA-digest of `SignedAttrs` |
