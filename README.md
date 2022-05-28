# BIPF spec

Specification for BIPF (Binary In-Place Format). A binary format
designed for in-place (without parsing) reads, with schemaless
json-like semantics.

## Format

Data are type-length-value (TLV) encoded in BIPF. The type and length
are packed into a [varint] called tag. The type is stored in the lowest
3 bits, and the length the higher bits.

Since a varint can store values up to 128 bits in a single byte,
values less than 16 bytes long have a one byte tag, and values up to
8k long have a two byte tag, values up to 1048576 bytes have a 3 byte
tag, and so on. See [varints in protobuf] for more information.

```
<tag: varint(encoding_length(value) << 3 | type)><value>
```

The type indicates the encoding of the value.

Valid types are:

```
STRING  : 0 (000) // utf8 encoded string
BUFFER  : 1 (001) // raw binary buffer
INT     : 2 (010) // little endian 32 bit integer
DOUBLE  : 3 (011) // little endian 64 bit float
ARRAY   : 4 (100) // sequence of any other value
OBJECT  : 5 (101) // sequence of alternating bipf encoded key and value
BOOLNULL: 6 (110) // a boolean, or null
EXTENDED: 7 (111) // custom type. Specific type should be indicated by varint at start of buffer
```

## Fixtures

This repo contains a `fixtures.json` with different examples. The json
and binary (bipf) values are hex encoded buffers.

## Motivation

For a detailed motivation, please refer to: https://github.com/ssbc/bipf#motivation.

## Implementations

 - JS: https://github.com/ssbc/bipf/
 - Go: https://git.sr.ht/~cryptix/go-exp/tree/bipf/item/bipf
 - Rust: https://github.com/jerive/bipf-rs


[varint]: https://en.wikipedia.org/wiki/LEB128#Unsigned_LEB128
[varints in protobuf]: https://developers.google.com/protocol-buffers/docs/encoding#varints
