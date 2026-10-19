# 3. Value formats

Three formats share one logical write order and differ in byte layout.
A socket MUST pick one. See [architecture](01-architecture.md).

## 3.1 Compressed

Compressed is the legacy opt-in. It is sequential. Frames, offset
checkpoints, and trailers write nothing.

| Operation | Bytes |
|-----------|-------|
| `u8` | 1 raw byte |
| `i8` | 1 byte, `value XOR 0x80` |
| `u16` / `u32` / `u64` | uvarint16 / uvarint32 / uvarint64 |
| `i16` / `i32` / `i64` | svarint16 / svarint32 / svarint64 |
| `bool` | 1 byte, `0` or `1` |
| `f32` / `f64` | sortable float |
| length, string, binary | uvarint32 length, then bytes |
| map | uvarint32 count, then interleaved key, value pairs |
| pointer | svarint32 delta; body deferred |
| typed value | uvarint32 type index, then body |

Pointer delta:

| Delta | Meaning |
|-------|---------|
| `0` | null |
| `< 0` | already written; index is `written_count + delta` |
| `> 0` | deferred; 1-based index in the later queue |

After the root value, deferred bodies are appended in queue order.
Each deferred body is a typed value: uvarint32 type index, then the
value. Both peers MUST register dynamic types in the same order.

## 3.2 Flat

Flat is the default for RPC and stream messages. Integers and floats
are fixed-width little-endian. The goal is one-pass write and random
access to variable fields.

### Frame

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         payload ...                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| format (u8)   | checkpoint 0 (W bytes, LE) ...                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| ... checkpoints ...                           | format (u8)   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

The trailer exists only when `K > 0`, where `K` is the number of
variable-field checkpoints. If `K` is 0 there is no trailer.

- `format` bits 0-1 select the checkpoint width: `0` means 1 byte,
  `1` means 2 bytes, `2` means 4 bytes.
- The same `format` byte is stored at both ends of the trailer so a
  reader can find the trailer from the end: trailer size is
  `2 + K * W`.
- A checkpoint is the write position relative to the start of the frame,
  stored little-endian in `W` bytes.
- A checkpoint is written after a string, binary, container, map, or
  user-defined type. It is NOT written after a scalar, an enum, a bool,
  or a `shared_ptr` handle (the handle is a fixed `i32`).

### Maps

A flat map is a struct of arrays, not interleaved pairs. Keys are
emitted in `memcmp` order, not insertion order. An empty map is a
length of 0 and nothing else.

### Pointers

A pointer handle is an `i32` little-endian delta, with the same `0` /
negative / positive meaning as compressed. Deferred bodies are later
flat frames. Each deferred body starts with a `u32` little-endian type
index (`FLAT_WIRE_VERSION` 2). The root value of a message does not
carry that per-frame type index; its type is the socket message type.

## 3.3 Key

Key is for database keys. `memcmp` on the blob MUST match the intended
sort. Lengths MUST NOT sit between data bytes.

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         data ...                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         meta ...                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| meta_size u8  |
+-+-+-+-+-+-+-+-+
```

- **data** holds scalars (compressed encodings, including sortable
  floats and `i8 XOR 0x80`) and the raw bytes of strings and binaries.
- **meta** holds uvarint32 lengths in the order they were written.
- **meta_size** is one byte, the size of meta, from 0 to 255.
  A writer MUST fail if meta would exceed 255 bytes.

A reader finds the boundary as `n = size - 1 - meta_size`. Data is
`blob[0, n)` and meta is `blob[n, size - 1)`.

Key MUST reject maps, shared pointers, typed values, messages, and
exceptions. Frames and pointer tails write nothing.

## 3.4 User-defined types

Field order is base classes first, then non-static data members, in
declaration order. Members marked ignore are omitted. Both peers MUST
use the same order.
