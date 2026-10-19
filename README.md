# rpc-protocol

Wire specification of the Daemonarium RPC protocol, version 0.1.0.

This repository is the definition. It is not an implementation.
[rpc-cpp](https://github.com/daemonarium/rpc-cpp) and
[rpc-ts](https://github.com/daemonarium/rpc-ts) (`@daemonarium/rpc`)
are the two implementations that conform to this version.

## Read it

Start at [spec/00-introduction.md](spec/00-introduction.md).

| Section | Subject |
|---------|---------|
| [1](spec/01-architecture.md) | Layers and the one-format-per-socket rule |
| [2](spec/02-primitives.md) | Integers, uvarint, sortable float |
| [3](spec/03-value-formats.md) | Flat, compressed, and key layouts |
| [4](spec/04-type-registry-blob.md) | Schema blob and its envelope |
| [5](spec/05-socket-framing.md) | Length prefix, sentinel, trailing payload |
| [6](spec/06-rpc-messages.md) | Request, response, error |
| [7](spec/07-stream-messages.md) | Observation streams |
| [8](spec/08-observable-state.md) | Receiver state machine |
| [9](spec/09-examples.md) | Annotated hex |
| [10](spec/10-conformance.md) | What an implementation must do |
| [11](spec/11-security.md) | What this layer does not protect |
| [12](spec/12-registry.md) | Assigned type indices |

Frame layouts are ASCII packet diagrams. State machines are Mermaid
(`diagrams/`). Vectors are in `test-vectors/`.

## Out of scope

The gateway session protocol (`api_message`, virtual sockets) is a
separate protocol. It is not specified here.

## License

MIT. See `LICENSE`.
