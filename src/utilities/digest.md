# Multi-Circuit Digest

The ability of digesting dynamically sized payloads using SHA algorithm is provided by the excellent [Mina Attestations](https://github.com/zksecurity/mina-attestations) library. We are using a fork to expose some of the needed functionality, using the same underlying code.

SHA digesting a long payload like `LDS` or a `x509` certificate do not fit into a single circuit. Therefore, we use a scheme of Update & Validate to calculate the digest value.

Poseidon-digest is way cheaper than SHA-digest in circuits.

> Note: we are carrying a `carry` field so that we can plug this in our pipeline, carrying the previous leaf's output.

#### Digest State Data Type

| Input        | Type                    |
| ------------ | ----------------------- |
| `len`        | SHA size                |
| `state`      | `StaticArray(UIntN, M)` |
| `commitment` | `Field`                 |
| `carry`      | `Field`                 |

### Update

For each `update` step, we are digesting a chunk of values, which is a couple of SHA blocks, at the same time with both SHA and Poseidon algorithms. Between each `update`, the digest state is always fed to the circuit.

Only the current state and the chunk is fed to a update circuit, not the whole payload.

#### Private Inputs

| Input          | Type                                            |
| -------------- | ----------------------------------------------- |
| `digest state` | `DigestState`                                   |
| `iteration`    | `StaticArray<BlockN>` or `DynamicArray<BlockN>` |

where iteration holds the `chunk` to be digested

#### Public Output

| Left                             | Right                                     |
| -------------------------------- | ----------------------------------------- |
| Poseidon-digest of `DigestState` | Poseidon-digest of the next `DigestState` |

### Validate

We give the whole payload as well as the latest digest state which contains both digest algorithms state inside, to the circuit.

The circuit Poseidon-digests the whole payload again to check if the previous steps processed the correct payload chunks in correct order. If all is well, it returns the SHA-digest output.

This step happens inside the "caller" circuit since on its own does not mean much, only meaningful alongside other validations from the application logic.

#### Required Inputs

| Input          | Type           |
| -------------- | -------------- |
| `digest state` | `DigestState`  |
| `payload`      | `DynamicBytes` |
| `carry`        | `Field`        |
