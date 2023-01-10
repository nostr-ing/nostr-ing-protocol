# nostr-ing: Transmitting non-text data on top of nostr

## Motivation

Traditional networks and social media rely on trust in central entities. Whether it is authentication, verification, storage, or distribution, they do everything today. The nostr protocol seeks to decentralize these networks. According to the protocols repository, nostr is:

>“The simplest open protocol that is able to create a censorship-resistant global "social" network once and for all. It doesn't rely on any trusted central server, hence it is resilient; it is based on cryptographic keys and signatures, so it is tamperproof; it does not rely on P2P techniques, therefore it works.” [^1]

Over the last two decades, social media has become more visual. One can easily witness this when observing the current nostr feeds. Because the protocol does not natively support the transmission of non-text images, users started to upload their media files on central servers and include links to those in their posts, leading to problems:

1. Hosting providers can change the file inside a URL without invalidating the note
2. Hosting providers can arbitrarily censor or shut down content

Instead, we need a way to transmit cryptographically secure non-text files to multiple relays.

## Transmitting non-text data on nostr

nostr excels at text data transmission. Clients and Relays transmit events as JSON-serialized strings. If we want to transmit an image to multiple relays, we need to encode it in JSON-safe format, introducing inefficiencies. For example, the Base64 encoding increases the required storage space by approximately 33% as Base64 requires an 8-bit character to store 6 bits of raw data. [^2]

These inefficiencies are particularly problematic because storage and bandwidth requirements directly affect the costs of running a nostr relay and, therefore, also the potential decentralization of relays in general.

## A second layer for binary data

The solution we propose to solve the problem outlined above is a new protocol that acts as a second layer to the nostr base layer. It is specifically designed to excel at the transmission and storage of binary data. WebSockets, the protocol that nostr is based on, already directly supports the transmission of raw binary data. Therefore we can reuse most of the infrastructure already available to standard nostr relays. When clients send or request a non-text file to or from a relay, they open a second connection that will transmit binary data only. As this second layer is optional to the base layer, not every nostr relay will be able to support these requests. Therefore, clients use WebSockets sub-protocol header “Sec-WebSocket-Protocol” to signal the requirement of a binary-only connection according to this proposal.

## Transmitting data

A file stream on nostr-ing is called a Transmission. Every transmission has two parts to it:
Data
Metadata

To maximize compatibility and lay the groundwork for continuous data (e.g., live audio streaming), data on nostr-ing is transmitted in chunks. We do not define a maximum chunk size and leave it to clients and relays to agree on. It is worth pointing out though, that the choice of chunk size can directly affect the potential decentralization of a transmission.
In addition to the chunked data, a client publishes a single event on the nostr base layer, acting as a metadata store and helping with transmission discovery.

### Data chunks

As described above, data on nostr-ing is divided into chunks. A data chunk is always structured like so:

`data-length (4 bytes) - data - sequence-number - transmission-id (32 bytes) - signature (32 bytes)`

`data-length`: The length of the data chunk inside this message in bytes.

`data`: The data chunk itself

`sequence-number`: The sequence number of this chunk. When splitting, a client will number all chunks starting with 0

`transmission-id`: The Id of this transmission. It will be the event Id of the metadata event. 

`signature`: A signature of the whole message according to Schnorr Secp256k1

### Metadata event

The metadata event is an entry point for a nostr-ing transmission and is unlike data chunks sent directly to the nostr base layer. It contains a transmission’s metadata and acts as a hinge point for all data chunks, as all file chunks contain it's event-id.

In order to avoid incompatible relays and clients storing this metadata, we propose a new kind `X`. Kind `X` is used to transmit all kinds of metadata about a transmission, but they need to carry at least these key-value pairs:

`transmission-length`: Total size of a transmission in bytes (for continuous data this is ‘*’)

`transmission-type`: Filetype transmitted according to MIME

`chunk-size`: Size of data chunks in bytes

## Receiving Data

Receiving clients subscribe to transmissions by connecting to a nostr-ing relay and passing a transmission ID as part of the connection URL.

## Storing Data

A Relay will store data chunks in two collections / tables.

1. Files
2. Data

### Files

The `Files` table stores all metadata events of kind `X` indexed by event-id. It can be queried to receive metadata about a transmission or to initialize one.

### Data

The `Data` table stores all data chunks indexed by their transmission-id (event-id) and sequence-number. It can be queried to receive all data chunks of a transmission or allow for range queries by specifying a sequence-number.


[^1]: [nostr](https://github.com/nostr-protocol/nostr)
[^2]: [RFC4648](https://www.ietf.org/rfc/rfc4648.txt)