# 10. Conformance

An implementation conforms to protocol 0.1.0 when every statement
below holds.

## Formats

1. The implementation MUST encode and decode the rows in
   [test-vectors](../test-vectors/) without remainder and without
   error.
2. The implementation MUST NOT accept a flat body on a socket that was
   opened as compressed, or the reverse.
3. Key encoding MUST reject maps, shared pointers, and typed values.
4. Flat graph subframes MUST include the `u32` little-endian type index
   (`FLAT_WIRE_VERSION` 2).

## Framing

5. The length prefix MUST be a little-endian `u32` and MUST count only
   the bytes that follow it.
6. On a flat socket the implementation MUST stop the message at the
   `u32` sentinel `0` and MUST treat the rest as trailing payload.
7. A void `rpc_response` MUST NOT contain a success flag byte.

## RPC

8. `request_id` in the response or error MUST equal `request_id` in
   the request.
9. Unknown `function_id` MUST produce `rpc_error`, not silence.
10. A client MUST NOT have two outstanding calls with the same
    `request_id` on one socket.

## Streams

11. `stream_item` for an id MUST NOT precede `stream_opened` for that
    id, except for producers that document a fast path which never
    emits `stream_opened`. A receiver of such a fast path MUST NOT
    require `stream_opened`.
12. Empty `stream_item.value` MUST be applied as a delete.
13. On `stream_reset` the receiver MUST discard items received since
    the start of the current delta.
14. `stream_cancel` for an unknown id MUST be ignored.
15. Index 5 MUST NOT be sent.

## Registry blob

16. Bit 7 clear means a legacy compressed body.
17. A non-zero reserved bit or an unknown format nibble MUST be
    rejected.

## Version

18. A conforming implementation MUST identify itself as protocol 0.1.0
    and `FLAT_WIRE_VERSION` 2. It MAY accept older inputs only where
    this document defines a legacy rule (the registry blob).
