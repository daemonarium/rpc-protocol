# 5. Socket framing

Every socket message is one length-prefixed envelope. The length counts
the bytes after the length prefix. It does not include itself.

## Flat (default)

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  length (u32 LE)                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               type index (u32 LE)                             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| encoded_size_0 = N0+1 (u32 LE)| root frame (N0 bytes)         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| encoded_size_i = Ni+1         | type index (u32 LE)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| deferred frame i (rest of Ni)                                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| 0 sentinel (u32 LE)           | trailing payload ...          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Rules:

- `encoded_size` is the real subframe size plus 1. Zero is the
  sentinel, so an empty subframe encodes as 1.
- The root frame has no per-frame type index. Its type is the message
  type index in the header.
- Each deferred subframe (pointer `write_rest`) is `encoded_size`, then
  a `u32` little-endian type index, then the flat frame. The real size
  counts the type index plus the frame.
- After the sentinel, the remaining bytes of `length` are the trailing
  payload. A dispatcher MUST NOT deserialize the trailing payload as
  part of the message struct.
- A receiver MUST walk subframes until the sentinel, then take
  `length` minus the cursor as the payload size.

## Compressed (opt-in)

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  length (u32 LE)                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| type index (uvarint)          | compressed body ...           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| deferred typed bodies ...     | trailing payload ...          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

There is no per-frame size prefix, no trailer, and no `u32` sentinel.
The type index is a uvarint. Deferred bodies are appended by the
compressed pointer rules. The trailing payload is whatever remains
after the body and the deferred bodies.

A peer MUST NOT put flat bodies and compressed bodies on the same
socket.
