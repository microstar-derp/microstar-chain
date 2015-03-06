#### If this documentation is at all unclear, please do not hestitate to file an issue. Ease of implementation is the primary goal of this project.

# microstar-chain
This module creates chains of messages. Each message (except for the first) contains the hash of the previous message. Each message is also signed. This allows microstar-chain to verify that a given chain is contigous and that it comes from a certain node.

## Conventions

#### `settings` must contain:
- `db` - This is a leveldb.
- `keys` - This is a keypair generated by `microstar-crypto`
- `index_defs` - This is an array of `level-librarian` index definitions, to be used when writing documents to the db.

### `index_defs`
This module requires some index definitions to be present on every document in the db. These index definitions are available as `mChain.index_defs`. Add these index definitions to the settings object that you pass to all other microstar modules.

## API

### mChain.write(settings, callback)
This method returns a `pull-stream` sink that writes a chain of messages to the db. The callback is called when all documents have been written to the db.

Messages streamed to `.write()` must have the following properties:

```json
{
    "content": "Fa", // Can be any valid JSON
    "type": "holiday-carols:syllable",
    "chain_id": "holiday-carols:2014",

    "timestamp": 1418804138168, // This is optional, if it does not exist
}
```

### mChain.writeOne(settings, doc, callback)
This method accepts a single document and a callback, calling the callback when the document has been written to the db.

### mChain.read(settings, query)
This method simply aliases `read` from `level-librarian`. It returns a `pull-stream` source that reads documents from the database according to a `level-librarian` query.

### mChain.readOne(settings, query, callback)
This method simply aliases `readOne` from `level-librarian`. Instead of returning a stream, it accepts a callback that is called once with the result of the query.

#### Messages
- `content` - The content of the message. This can be any JSON.
- `type` - The type of the message. Please follow a convention of prefixing the message type with the name of your module like this: `<name of module>:<type of message>`.
- `chain_id` - An identifier for the chain. Namespace this in the same way as `type`.
- `previous` - The hash of the previous message in the chain.
- `public_key` - The public key of the creator of the message.
- `sequence` - The oprder of the message in the chain.
- `signature` - A signature of the message contents, created with the `public_key`.
- `timestamp` - A timestamp of when the message was created.


### mChain.validate(settings, initial, callback)
- `initial` - This is the message that evaluation starts with.

This method returns a through stream that takes a stream of sequential, formatted messages and validates them using the `initial` message as the first message in the stream. Stream errors on the first invalid message.

```js
var mChain = require('microstar-chain')

pull(
  pull.values([{ // Messages in the stream
    chain_id: 'holiday-carols:2014',
    content: 'La',
    previous: 'LWTQmsJ1E9fu+gSXDM03ckBXieL9/K8Jl2claIRcC6FFX5WYd1ojDsgo6KK1GafCinq2lAQlsIeVtU4RSpYL1w==',
    public_key: 'N3DyaY1o1EmjPLUkRQRu41/g/xKe/CR/cCmatA78+zY=7XuCMMWN3y/r6DeVk7YGY8j/0rWyKm3TNv3S2cbmXKk=',
    sequence: 1,
    signature: '/v1TqoggUpzuFx5sJ5jirlQsBOpGQBb1DJwP4ue1S5LzqKXIvZvlFe/WOLjyQTKXkqw9uQo2NH7eJPq4E7HbAQ==',
    timestamp: 1418804138169,
    type: 'holiday-carols:syllable'
  }, {
    chain_id: 'holiday-carols:2014',
    content: 'Laa',
    previous: '31+k7zPSRtH22OZxA4RXQRNQJ42gay0LNcGSUt19JhS/RElqw/O28+eRUQQdKJvSiQNjU1I5hyHf9OG7I1Np3g==',
    public_key: 'N3DyaY1o1EmjPLUkRQRu41/g/xKe/CR/cCmatA78+zY=7XuCMMWN3y/r6DeVk7YGY8j/0rWyKm3TNv3S2cbmXKk=',
    sequence: 2,
    signature: '+2r2xOcwEsP/h2inzDYx3OX2jk+03Zjnhp7pdagNcDFAE/fhdTX4Zmdx+Vi+divPumjIvHQYNSzy4qBI9c4dAQ==',
    timestamp: 1418804138170,
    type: 'holiday-carols:syllable'
  }]),

  mChain.validate({ // Initial message
    chain_id: 'holiday-carols:2014',
    content: 'Fa',
    previous: null,
    public_key: 'N3DyaY1o1EmjPLUkRQRu41/g/xKe/CR/cCmatA78+zY=7XuCMMWN3y/r6DeVk7YGY8j/0rWyKm3TNv3S2cbmXKk=',
    sequence: 0,
    signature: 'Cs8s0zZgqE/Tp+DCFDXuMYA6mNUtPTFGf//5rENPCx37g3L7BFhz0pBJ06GFK5E1i3C6o5H9BgX/Ltppf5EFBQ==',
    timestamp: 1418804138168,
    type: 'holiday-carols:syllable'
  }),

  pull.log()
)

```

### mChain.copy(settings, initial, callback)

- `initial` - Passed through to `validate`

This writes a stream of sequential, formatted messages to the db, running them through `validate` first.