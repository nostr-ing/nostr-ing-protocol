#nostr-ing - A second layer protocol for streaming non-text data on nostr

nostr-ing is a second layer to the nostr-protocol that allows transmission of non-text data by Relays on top of the base protocol.

nostr-ing is a work-in-progress!

## A brief introduction to nostr

From the nostr-protocol repository:

>The simplest open protocol that is able to create a censorship-resistant globall "social" network once and for all. It doesn't rely on any trusted central server, hence it is resilient; it is based on cryptographic keys and signatures, so it is tamperproof; it does not rely on P2P techniques, therefore it works.

## Why have a second layer?

Nostr is awesome, but it was built to excel at handling text. Sending other file-types introduces inefficiencies due to encoding. Therefore we propose a second protocol that operates on top of nostr and that is optimised to transmit raw data chunks.

## How does it work?

### Connecting

nostr-ing connections between clients do not replace the original nostr connection, but are complementary. Clients will establish a connection on the no-string protocol only when receiving or sending non-text data and close them once they are done.

Clients will ask Relays for a nostr-ing connection by adding "Sec-WebSocket-Protocol: nostr-ing" to the handshake. The client will also specify a unique file-Id when connecting to a Relay by adding it as a query parameter to the connection string.

#### file-ids

File-ids are 128-bit UUIDs that are used to identify transmissions and act as a "room" for the transmission. Every client involved in a transmission (either broadcasting or listening) attaches the file-id to the connection string.

### Discovery

File transmissions can be published to the nostr network by transmitting a note that contains the file-id.

### Sending Data

On the nostr-ing protocol, data is sent as binary chunks. A events payload is always structured  as follows:

4 Bytes - Length of data in bytes
X Bytes - Data
X Bytes - Metadata

There is no maximum payload length specified, but note that some relays might choose to reject large payloads.

#### Metadata

Nostr-ing payloads always include metadata, such as the Content-Range, the Senders pubkey and a signature. Metadata is appended to the end of the payload. 


### Signing Data

In order to stop Relays or MIM from modifying data, its origin and integrity must be verifiable by every client. An easy solution for verifying files would be to hash them and publish a singed message of a files hash on nostr.

However ==nostr-ing== aims to allow the transfer of files, as well as live data, and therefore every single sent data chunk must be signed. The signature is part of the metadata that is appended to the end of the payload.

### Authentication

Before a Client can transmit data to a Relay it needs to authenticate. A Relay will challenge a Client on connection to sign a random piece of data. The Client will create a Schnorr Signature over the secp256k1 curve (BIP340) of the challenge and send it back to the Relay.

### Managing permissions

In order to facilitate not only one-to-many broadcasts, but also many-to-many ones (e.g., Twitter Spaces), permissions in nostr-ing rooms can be managed by room admins. By default only the client opening a room is an admin and can set permissions for other clients, by transmitting a set-permission message 

## Message Types

In order to do different things on the network, different types of messages are used. A type is indicated by the first byte of a messages payload.

### Authenticate

### Transmit Data

### Set Permissions
