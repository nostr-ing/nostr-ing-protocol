# nostr-eam

nostr-eam is a second layer to the nostr-protocol that allows transmission of non-text data by Relays on top of the base protocol. 

## A brief introduction to nostr

From the nostr-protocol repository:

>The simplest open protocol that is able to create a censorship-resistant globall "social" network once and for all. It doesn't rely on any trusted central server, hence it is resilient; it is based on cryptographic keys and signatures, so it is tamperproof; it does not rely on P2P techniques, therefore it works.

## Why have a second layer?

Nostr is awesome, but it was built to excel at handling text. If we would try to send non-text data over the protocol the encoding specified would introduce inefficiencies. Therefore we propose a second protocol that operates on top of nostr and that is optimised to transmit raw data chunks.
