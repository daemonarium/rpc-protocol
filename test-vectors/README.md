# Test vectors

Normative byte strings for protocol 0.1.0. A conforming encoder MUST
produce these hex strings. A conforming decoder MUST recover the value
column and consume every byte.

| File | Contents |
|------|----------|
| `compressed-scalar.tsv` | compressed scalars |
| `flat-scalar.tsv` | flat little-endian scalars |
| `flat-complex.tsv` | Person values and graphs, both compressed and flat |

Columns are separated by tabs: type name, input value, hex. In
`flat-complex.tsv` the columns are case name, format, seed, hex.

These files are the vectors checked in with
[daemonarium/rpc-ts](https://github.com/daemonarium/rpc-ts) v0.1.0
(`tests/serialization_*_reference.tsv`). The C++ generators that
produce them ship in
[daemonarium/rpc-cpp](https://github.com/daemonarium/rpc-cpp) v0.1.0
(`generate_serialization_tsv`, `generate_serialization_complex_tsv`).
Regenerate only when the protocol version changes, then commit the
new bytes in the same change as the spec.
