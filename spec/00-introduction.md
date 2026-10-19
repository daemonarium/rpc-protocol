# 0. Introduction

This document is the wire specification of the Daemonarium RPC protocol,
version 0.1.0. It defines bytes on the wire. It does not define an API,
a programming language, or a transport.

Two implementations claim conformance to this version:

- [daemonarium/rpc-cpp](https://github.com/daemonarium/rpc-cpp) v0.1.0
- [daemonarium/rpc-ts](https://github.com/daemonarium/rpc-ts) v0.1.0,
  published as `@daemonarium/rpc`

## Requirements language

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD,
SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as
described in BCP 14 ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119),
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when,
they appear in all capitals.

## Versioning

The protocol version is independent of any implementation version.
`FLAT_WIRE_VERSION` is 2. It names the flat graph layout in which each
deferred subframe carries a little-endian `u32` type index.
A sender MUST NOT emit `FLAT_WIRE_VERSION` 1 graphs (no per-frame type
index) to a peer that requires version 2. A receiver that requires
version 2 MUST reject a version 1 graph before exchanging pointers.

A wire-incompatible change requires a new protocol version. Adding a
type index past 10 is compatible if both peers agree on registration
order for application types. Changing the meaning of indices 0 through
10 is not compatible.

## Scope

In scope:

- primitive encodings
- three value formats: flat, compressed, and key
- the type-registry blob
- socket message framing
- RPC messages `rpc_request`, `rpc_response`, and `rpc_error`
- stream messages and the observable state machine

Out of scope:

- the gateway session protocol (`api_message`, `mux_socket_*`)
- transport security, authentication, and authorization
- schema evolution
- programming-language APIs

## Terminology

- **Value format.** The encoding of one serialized value: flat,
  compressed, or key.
- **Frame.** One framed value, including any trailer the format adds.
- **Socket message.** A length-prefixed envelope that carries a typed
  message and an optional trailing payload.
- **Type index.** A small integer that names a message or dynamic type.
  Indices 0 through 10 are assigned by this specification.
  Application types start after those assignments.
- **Trailing payload.** Bytes after the message body and after any
  deferred pointer bodies. Arguments, results, and exceptions travel
  here. They are not fields of the message struct.
