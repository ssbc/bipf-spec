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
ATOM    : 6 (110) // 1 = true, 0 = false, no value means null.  Other values are for application purposes.  
EXTENDED: 7 (111) // custom type. Specific type should be indicated by varint at start of buffer
```

## About 'ATOM' type
In addition to representing boolean values and `null`, the ATOM type in BIPF can also be used to store application-specific symbols or dictionary-mapped values. These custom values can be any 32-bit unsigned integer within the range of the encoded value. This makes the ATOM type a flexible option for encoding a wide range of data types, including custom symbols or values that are specific to a particular application or use case.

By allowing for the encoding of custom symbols or values, the ATOM type can also help reduce the size of BIPF-encoded data by replacing longer strings or values with shorter, more efficient representations. This can lead to faster data processing times and reduced storage requirements, making BIPF a practical choice for applications where efficient data storage and processing are important.

It is worth noting that the ATOM type uses a little endian encoding scheme, which means that the least significant bytes are stored first in the encoded value. Additionally, any least significant null bytes except last one in the encoded value are trimmed, resulting in a more compact representation of the data. This little endian encoding scheme and trimming of null bytes can help further reduce the size of BIPF-encoded data, making it even more efficient for storage and processing.

Examples:
- 00000110 -> `null`
- 00001110 00000000 -> `false`
- 00001110 00000001 -> `true`
- 00001110 00000010 -> atom number 2 for the application
- 00010110 00000000 00000001  -> atom number 256 for the application





## About 'OBJECT'
When encoding an OBJECT type in BIPF, the keys and values are encoded in an alternating sequence, starting with the key, where each key is either a STRING or an ATOM type. This means that the first element in the encoded sequence is always a key, followed by its corresponding value, followed by the next key, and so on.

If an ATOM key is used, it is the responsibility of the application to maintain a mapping between the encoded ATOM values and their corresponding STRING keys. This can be useful for encoding data where the keys are limited to a small set of predefined values or symbols. However, it is important to avoid using the predefined values of `null`, boolean `false`, and boolean `true` as keys in order to avoid conflicts with their predefined meanings as values in the context of an ATOM type.



## Fixtures

This repo contains a `fixtures.json` with different examples. The json
and binary (bipf) values are hex encoded buffers.

## Motivation

For a detailed motivation, please refer to: https://github.com/ssbc/bipf#motivation.

## Implementations

 - JS: https://github.com/ssbc/bipf/
 - Go: https://git.sr.ht/~cryptix/go-exp/tree/bipf/item/bipf
 - Rust: https://github.com/jerive/bipf-rs
 - Nim: https://github.com/BundleFeed/nim_bipf (npm package with JS compiled version: https://www.npmjs.com/package/nim_bipf)


[varint]: https://en.wikipedia.org/wiki/LEB128#Unsigned_LEB128
[varints in protobuf]: https://developers.google.com/protocol-buffers/docs/encoding#varints
