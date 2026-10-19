# 12. Type index registry

Indices 0 through 10 are assigned by this specification. Both peers
MUST register them in this order before any application type.
Application type indices begin at 11.

Changing an assigned index is a new protocol version.

| Index | Type | Direction | Body |
|-------|------|-----------|------|
| 0 | `rpc_request` | client to server | `request_id`, `function_id`, trailing arguments |
| 1 | `rpc_response` | server to client | `request_id`, trailing result |
| 2 | `rpc_error` | server to client | `request_id`, trailing exception |
| 3 | `socket_opened` | server to client | empty |
| 4 | `stream_item` | server to client | `stream_id`, `key`, `value`, `timestamp` |
| 5 | `stream_key` | reserved | not sent in 0.1.0 |
| 6 | `stream_done` | server to client | `stream_id` |
| 7 | `stream_error` | server to client | `stream_id`, `message` |
| 8 | `stream_reset` | server to client | `stream_id` |
| 9 | `stream_cancel` | client to server | `stream_id` |
| 10 | `stream_opened` | server to client | `stream_id` |

Index 3 is `socket_opened` here. A session protocol that puts a
different message at index 3 is specified separately and is not this
registry.

## Data format nibble

Assigned by [the type registry blob](04-type-registry-blob.md).

| Nibble | Name |
|--------|------|
| 0 | compressed |
| 1 | flat |
| 2 | key |
| 3-15 | unassigned; receivers MUST reject |

## Stream flags

| Value | Name |
|-------|------|
| 0 | `none` (finite scan) |
| 1 | `observe` (live) |
| 2-255 | unassigned; receivers MUST reject |
