# nostr-ing

nostr-ing is a second layer to the nostr-protocol that allows transmission of non-text data by Relays on top of the base protocol. 

## A brief introduction to nostr

From the [nostr-protocol](/nostr-protocol/nostr) repository:

>The simplest open protocol that is able to create a censorship-resistant globall "social" network once and for all. It doesn't rely on any trusted central server, hence it is resilient; it is based on cryptographic keys and signatures, so it is tamperproof; it does not rely on P2P techniques, therefore it works.

## Why have a second layer?

Nostr is awesome, but it was built to excel at handling text. Sending other file-types introduces inefficiencies due to the non-text data over the protocol the encoding specified would introduce inefficiencies. Therefore we propose a second protocol that operates on top of nostr and that is optimised to transmit raw data chunks.

## How does it work?

### Connecting

When opening a connection to a Relay, a Client with ==nostr-ing== support will open two concurrent connections. The first for the base protocol and the second one for the ==nostr-ing== sub-protocol. If a Relay does not support the sub-protocol, only with first connection succeeds.

### Sending Data

Data on the ==nostr-ing== sub-protocol is sent as raw data chunks. This allows for a much more efficient transfer (33% compared to base64).

### Signing Data

In order to stop Relays or MIM from modifying data, its origin and integrity must be verifiable by every client. An easy solution for verifying files would be to hash them and publish a singed message of a files hash on nostr.

However ==nostr-ing== aims to allow the transfer of files, as well as live data, and therefore every single sent data chunk must be signed. Clients shall therefore reserve the last 64 bytes of every transmission for a signature of the payload.

Example (4096 byte chunk):

2 bytes of prefix - 4030 bytes of data - 64 byte signature

### Authentication

Before a Client can transmit a file to a Relay it needs to authenticate. A Relay will challenge a Client on connection to sign a random piece of data. The Client will create a Schnorr Signature over the secp256k1 curve (BIP340) of the challenge and send it back to the Relay.


