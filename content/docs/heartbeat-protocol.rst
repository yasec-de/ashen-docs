---
title: "Heartbeat Protocol Specification"
---

Protocol Stack
--------------

The ashen heartbeat protocol serves multiple purposes. It enables the payload system and the tamper sensor to ascertain
each other's cryptographic identity. Further, it allows the payload to continuously monitor the tamper sensor's state to
ensure secrets within the payload are quickly deleted when the sensor detects a tampering attempt. Finally, it serves as
the cryptographically secured transport layer for any higher level cryptographic functions such as key management
services that the tamper sensor may provide to the payload.

The protocol is canonically spoken over a UART serial connection. This choice was made since for the protocol's intended
use, larger bandwidth than an UART can provide is not needed, and an UART is maximally compatible with microcontrollers
and other platforms that could be used as the base of a tamper sensor. The canonical UART settings for the protocol are
8N1 framing at 1 MBd speed.

On top of the raw UART interface, the protocol uses COBS to provide framing. Packets are encoded using COBS, and packets
must be terminated by a null byte. The terminating null byte must be sent as soon as the COBS-encoded packet is sent,
and should not be delayed until the next packet is sent. Inserting additional null bytes in between packets is
acceptable as it may be useful for synchronization.

On top of COBS, the protocol uses the `Noise protocol`_ using the ``XX`` pattern. The chosen cryptographic primitives
are Curve25519 for Diffie-Hellman operations, ChaCha20/Poly1305 for Authenticated Encryption with Associated Data
(AEAD), and Blake2s for hashing both when used as a bare hash by the protocol and when used through Hash-Based Key
Derivation Function (HKDF). The name of this Noise protocol according to the `Noise protocol specification`_ is
``Noise_XX_25519_ChaChaPoly_BLAKE2s``.

{{< hint info >}}
    Right now, the protocol is not post-quantum secure as it relies on Noise's standard Diffie-Hellman primitive using
    the X25519 elliptic curve. A transition to <a href="https://eprint.iacr.org/2022/539">Post Quantum
    Noise (PQNoise)</a> is planned for a later protocol revision.
{{< /hint >}}

The ``XX`` Noise pattern has both parties contribute an ephemeral key as well as a static long-term identity key to the
connection. In this pattern, both parties transmit their static long-term identity public keys to each other, enabling
trust on first use (TOFU) without the need to transmit the keys out of band. Note that despite this, the protocol is
still identity hiding to a passive observer. Because ephemeral keys are used, the protocol provides forward secrecy from
one handshake to the next.

The heartbeat protocol does not transmit any encrypted payload data or any associated data during the Noise protocol
handshake. Only once the handshake is complete, does the heartbeat protocol start exchanging messages. The heartbeat
protocol maps one heartbeat protocol message to one noise protocol message. After every transmitted message in either
direction, the corresponding direction key's ``REKEY`` function is called as described in the Noise specification.

.. _`Noise protocol`: https://noiseprotocol.org/noise.html
.. _`Noise Protocol specification`: https://noiseprotocol.org/noise.html
