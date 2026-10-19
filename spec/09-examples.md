# 9. Examples

Vectors live in [test-vectors](../test-vectors/). Each row is
`type`, `value`, `hex`. Hex is the value encoding only, not a socket
frame. Implementations MUST match these bytes.

## Compressed `u64`

From [compressed-scalar.tsv](../test-vectors/compressed-scalar.tsv).

| Value | Hex | Why |
|-------|-----|-----|
| `0` | `00` | one byte, prefix `0` |
| `127` | `7f` | still one byte (`< 0x80`) |
| `128` | `8080` | two bytes, prefix `10` |
| `16383` | `bfff` | last two-byte value (`< 0x4000`) |
| `16384` | `c04000` | three bytes, prefix `110` |

## Flat `u64`

From [flat-scalar.tsv](../test-vectors/flat-scalar.tsv). Eight
little-endian bytes.

| Value | Hex |
|-------|-----|
| `0` | `0000000000000000` |
| `1` | `0100000000000000` |
| `42` | `2a00000000000000` |
| `128` | `8000000000000000` |

The same integer is one or two compressed bytes and always eight flat
bytes. A peer that mixes the two on one socket will mis-parse the
length of every following field.

## Empty user-defined value

`person_empty` in
[flat-complex.tsv](../test-vectors/flat-complex.tsv), seed `0x0`.

Compressed hex `00800080000000000000` is a sequence of zero-length
strings (uvarint `00` / `80` for the signed and length fields the
fixture writes). Flat hex begins `00000000` (a `u32` length of 0) and
ends with a trailer because the value has variable fields. The two
hex strings are not prefixes of each other.

## Registry envelope

A registry whose data format is flat, with an empty illustration of
the prefix only:

```text
81
```

Bit 7 is set (envelope present), bits 4-6 are clear, nibble is `1`
(flat). The compressed body follows. `80` would mean compressed data.
`82` would mean key data. A buffer whose first byte is `01` is legacy:
no envelope, body starts at byte 0, data format is compressed.
