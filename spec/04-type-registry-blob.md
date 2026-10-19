# 4. Type registry blob

A type registry blob is what a WebAssembly module returns from
`rpc_description`, and what a host stores to remember that module's
schema.

```text
 0
 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+
|   envelope    |  compressed type_registry body ...
+-+-+-+-+-+-+-+-+
```

The body is always the compressed encoding of the registry. The
envelope describes the format of the **data** that this registry
describes (messages, rows, index keys). It does not describe the
registry body.

## Envelope byte

| Bits | Meaning |
|------|---------|
| 7 | `1` means the envelope is present. `0` means a legacy blob: the entire buffer is the body. |
| 4-6 | Reserved. MUST be `0`. |
| 0-3 | Data format nibble. |

| Nibble | Name |
|--------|------|
| `0` | compressed |
| `1` | flat |
| `2` | key |

Examples: compressed `0x80`, flat `0x81`, key `0x82`.

A new registry's default data format is flat.

## Legacy

If bit 7 of the first byte is clear, the receiver MUST treat the whole
buffer as a compressed body and MUST assume the data format is
compressed.

## Errors

A receiver MUST reject an envelope whose reserved bits are not zero,
and MUST reject a format nibble other than `0`, `1`, or `2`.
