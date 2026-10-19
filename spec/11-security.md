# 11. Security considerations

This protocol does not authenticate peers and does not encrypt bytes.
A transport that needs either property MUST provide it. TCP, a
WebSocket, and a WebAssembly heap are equally unprotected at this
layer.

## Untrusted input

A receiver MUST bound the length prefix before allocating. A length
that exceeds the receiver's buffer policy MUST be rejected. A receiver
MUST bound uvarint length so a hostile prefix cannot request more
bytes than the variant allows.

Type indices, pointer deltas, and checkpoint offsets are attacker
controlled when the peer is untrusted. A receiver MUST reject a delta
or checkpoint that points outside the message. A receiver MUST reject
a type index that was not registered.

## Graphs

Pointer graphs can contain cycles. A receiver MUST use the delta rules
(already written versus deferred) and MUST NOT recurse without a
visited set. A deferred queue that does not end is a malformed message.

## Errors as data

`rpc_error` and `stream_error` carry a string from the peer. An
application MUST treat that string as data, not as a local control
message.

## No downgrade

A legacy registry blob (bit 7 clear) is compressed by definition.
A peer MUST NOT use that rule to force a flat socket into the
compressed format.
