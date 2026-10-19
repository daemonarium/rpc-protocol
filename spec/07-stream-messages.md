# 7. Stream messages

A stream is a sequence of messages that share one `stream_id`. The
client starts it by an RPC whose first argument is a `stream_handle`.
The producer then emits stream messages on the same socket.

`stream_observe_begin` and `stream_retransmit` are named in
implementations but are not registered in version 0.1.0. A sender
MUST NOT emit them. Assigning them an index is a future protocol
version.

## `stream_handle`

This struct is an RPC argument, not a socket message.

| Field | Type | Meaning |
|-------|------|---------|
| `pid` | `u64` | return process, transport-defined |
| `socket` | `u32` | return socket |
| `stream_id` | `u64` | id for later stream messages |
| `since_timestamp` | `u64` | `0` means a full snapshot; `> 0` means entries with timestamp greater than this value |
| `flags` | `u8` | `0` finite scan; `1` live observe |

`flags` and `since_timestamp` are orthogonal. An observe stream MAY
request a delta window. A scan MAY request a full snapshot.

`stream_flags`:

| Value | Name | After catch-up |
|-------|------|----------------|
| `0` | `none` | producer sends `stream_done` and the stream ends |
| `1` | `observe` | `stream_done` ends the catch-up; the stream stays open until `stream_cancel` or `stream_error` |

## `socket_opened` (index 3)

Direction: server to client. Empty body. This is the ack that a private
channel is bound. It has no `stream_id` and is not part of the
per-stream lifecycle.

Index 3 is `socket_opened` in this protocol. A gateway session that
reuses index 3 for a different message is a different protocol and is
not conformant to this registry.

## `stream_item` (index 4)

Direction: server to client.

| Field | Type |
|-------|------|
| `stream_id` | `u64` |
| `key` | binary |
| `value` | binary |
| `timestamp` | `u64` |

An empty `value` (length 0) is a tombstone: delete that key. A receiver
MUST apply last-timestamp-wins per key. Timestamp order across keys is
not significant.

## `stream_key` (index 5)

Reserved. Version 0.1.0 senders MUST NOT emit `stream_key`. The index
is assigned so later indices stay stable. A keys-only tombstone is a
`stream_item` with an empty value, not a `stream_key`.

| Field | Type |
|-------|------|
| `stream_id` | `u64` |
| `key` | binary |
| `timestamp` | `u64` |

## `stream_done` (index 6)

Direction: server to client. Field: `stream_id` (`u64`).

For `flags = none`, this ends the stream. For `flags = observe`, this
ends the catch-up and the stream stays open.

## `stream_error` (index 7)

Direction: server to client.

| Field | Type |
|-------|------|
| `stream_id` | `u64` |
| `message` | string |

The error is recoverable. It does not by itself destroy the
observation. See [observable state](08-observable-state.md).

## `stream_reset` (index 8)

Direction: server to client. Field: `stream_id` (`u64`).

The producer invalidated a delta in flight and restarts a full stream
on the same id. The receiver MUST:

1. discard delta items received before this message
2. apply the following full-stream items with last-timestamp-wins
3. treat keys that were in range but are absent from the full stream
   as deleted

## `stream_cancel` (index 9)

Direction: client to server. Field: `stream_id` (`u64`).
Fire-and-forget. A producer MUST ignore an unknown or already finished
id. Cancel is valid after `stream_opened`. A producer SHOULD also
accept a cancel that arrives before the stream is attached, and then
MUST NOT start that id.

## `stream_opened` (index 10)

Direction: server to client. Field: `stream_id` (`u64`).

The producer has the id in its map. The client MAY cancel or use the
stream. Items for that id MUST NOT be sent before `stream_opened`.

## Order

```text
start RPC
  -> stream_opened
  -> stream_item ...
  -> stream_done | stream_error | stream_reset
```

`stream_reset` is followed by a new full sequence of items and then
`stream_done`, on the same `stream_id`.
