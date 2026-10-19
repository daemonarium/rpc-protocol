# 2. Primitives

## Endianness

Fixed-width integers and IEEE 754 floats in the flat format are
little-endian. The key format uses the same scalar encodings as the
compressed format. Inside a compressed uvarint or svarint, the payload
bits are big-endian.

## Fixed-width integers (flat)

| Name | Width | Layout |
|------|-------|--------|
| `u8` / `i8` | 1 | raw byte; `i8` is two's complement |
| `u16` / `i16` | 2 | little-endian |
| `u32` / `i32` | 4 | little-endian |
| `u64` / `i64` | 8 | little-endian |
| `f32` / `f64` | 4 / 8 | IEEE 754 little-endian |
| `bool` | 1 | `0` or `1` |

A flat `bool` MUST be `0` or `1`. A receiver MUST reject any other byte.

## uvarint

An unsigned integer of variable length. The first byte encodes the
length in leading one-bits, then the value occupies the remaining bits
of that byte and the following bytes, big-endian.

uvarint64 length bands:

| Value range | Bytes | First-byte prefix |
|-------------|-------|-------------------|
| `< 0x80` | 1 | `0xxxxxxx` |
| `< 0x4000` | 2 | `10xxxxxx` |
| `< 0x200000` | 3 | `110xxxxx` |
| `< 0x10000000` | 4 | `1110xxxx` |
| `< 0x800000000` | 5 | `11110xxx` |
| full 64 bits | 9 | `11111111` plus 8 bytes |

uvarint32 is at most 5 bytes. uvarint16 is at most 3 bytes. The same
prefix rules apply. A receiver MUST fail if the encoded length exceeds
the variant's maximum.

## svarint

A signed integer of variable length.

- The most significant bit of the first byte is the sign: `1` means
  non-negative, `0` means negative.
- Length is leading ones for a non-negative value, or leading zeros
  followed by a one for a negative value.
- A negative magnitude is stored as the bitwise complement of the
  absolute value in the data bits.

Pointer deltas in the compressed format are svarint32.

## Sortable signed byte

In the compressed and key formats, `i8` is stored as `value XOR 0x80`
so that unsigned byte order matches numeric order.

## Sortable float

Compressed and key floats are a fixed 4 or 8 bytes, transformed so that
lexicographic byte order matches numeric order, then stored big-endian.

For `f64` (and `f32` on 32 bits):

1. Read the IEEE bits as an unsigned integer of the same width.
2. If the sign bit is set, bitwise-complement the word. Otherwise XOR
   the word with the sign bit.
3. Positive zero is stored as the word whose only set bit is the sign
   bit (`0x8000…`).
4. Write the word big-endian.

A receiver MUST invert this transform. NaN values compare equal as
bytes: every NaN of a given width encodes to the same byte string.

## Lengths and bytes

| Format | String / binary / container length |
|--------|--------------------------------------|
| flat | `u32` little-endian, then that many bytes or elements |
| compressed | uvarint32, then that many bytes or elements |
| key | length goes to the meta region; raw bytes go to data |

Strings are raw bytes. This specification does not require UTF-8
validation.
