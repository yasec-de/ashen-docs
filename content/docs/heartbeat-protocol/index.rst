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

.. figure:: Ashen\ Protocol\ Stack.drawio.svg
   :alt: Protocol stack diagram with four layers from text below
   :width: 20em
   
   The Ashen Protocol Stack

The protocol is canonically spoken over a UART serial connection. This choice was made since for the protocol's intended
use, larger bandwidth than an UART can provide is not needed, and an UART is maximally compatible with microcontrollers
and other platforms that could be used as the base of a tamper sensor. The canonical UART settings for the protocol are
8N1 framing at 1 MBd speed.

On top of the raw UART interface, the protocol uses COBS to provide framing. Packets are encoded using COBS, and packets
must be terminated by a null byte. The terminating null byte must be sent as soon as the COBS-encoded packet is sent,
and should not be delayed until the next packet is sent.

Empty COBS frames (i.e. two null bytes immedately after one another) are acceptable and are silently ignored. When a
sequence of at least 24 null bytes in a row is observed, this sequence is interpreted as a *protocol reset* and the
noise and application protocol layers are reset to their power-on state. An active noise connection is discarded, new
ephemeral keys are generated, and the application-layer peer identity is reset.

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
protocol maps one heartbeat protocol message to one noise protocol message.

Higher-Level Protocol
---------------------

.. figure:: protocol\ state\ machine.drawio.svg
   :alt: State machine state diagram of the ashen firmware.
   :width: 50em
   
   Ashen firmware protocol state diagram. Normal operating flow highlighted in green, initial setup in blue, error paths
   in red.

The high-level ashen heartbeat protocol consists of typed messages exchanged through noise-encrypted COBS frames. The
diagram above shows the firmware protocol state machine starting from desynchronized (power up) state.

During normal protocol operation, the states and transitions highlighted in green are traversed. When the host connects
to the firmware for the first time, the pairing sequence highlighted in blue is executed after the noise link has been
established. The pair command in this sequence makes the firmware store the host's long-term (static) identity key and
mark the connection trusted. On the next connection, the firmware will recognize the host during the handshake. In this
case, when the host requests the device's status after the handshake is done, the device will respond with both the
``is_keyed`` and ``peer_trusted`` flags set. ``is_keyed`` indicates that the firmware has trusted *some* host, and is
not just unprogrammed. ``peer_trusted`` indicates that the firmware has recognized the host's long-term identity key and
that it will respond to other protocol commands.

If the firmware does not recognize the host's long-term identity key during the handshake, it will still allow the
handshake to proceed, but it will respond with ``peer_trusted`` unset and will only accept ``erase``, ``status`` and
``disconnect`` commands. To recover access to such a device, the host can ``erase`` it, destroying all key material
inside as well as unlinking the previous host key. After the ``erase`` is complete, the device returns to ``Unpaired``
state from which the host can then issue the ``pair`` command.

The diagram below shows the normal message flow during an ashen connection. The host starts sending heartbeat requests
as soon as the protocol's connection phase has been completed. Like most other messages, heartbeat requests are always
initiated by the host. The host is responsible for scheduling them.

.. figure:: protocol\ sequence\ diagram.drawio.svg
   :alt: UML-style sequence diagram of the communication between the ashen host kernel driver and the ashen firmware.
   :width: 50em
   
   Ashen protocol sequence diagram

.. _`Noise protocol`: https://noiseprotocol.org/noise.html
.. _`Noise Protocol specification`: https://noiseprotocol.org/noise.html
