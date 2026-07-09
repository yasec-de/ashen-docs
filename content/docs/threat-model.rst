---
title: "Threat model"
---

System Description
------------------

This page contains a semi-formal analysis of the security of a *typical* Ashen-based HSM. The HSM we describe here is a
generic setup based on a *typical* Linux Single-Board Computer (SBC) running Ashen on Linux that is connected to a Yanartas
tamper sensor. You can use this threat model as a starting point for tailoring a more comprehensive threat model
covering your particular use of Ashen.

The following parts are in scope for this analysis:

1. The security of the Ashen-equipped kernel itself.
2. The security of the Heartbeat link protocol and its practical implementation.
3. The security of the Ashen sensor firmware.
4. The combined system security of the kernel and sensor components.
5. The operating lifecycle phase as defined below, which starts at the initial arming of the tamper sensor at the end of
   manufacturing, and ends with the destruction of `Critical Assets`_ on either a tamper alarm or after a
   decommissioning command.

The following parts are out of scope for this analysis:

1. The physical tamper sensing security of the Yanartas tamper sensor. Different Yanartas variants provide different
   levels of security, and protect against different classes of attacks.
2. The physical and logical security of the Linux SBC hardware and the integration between the Linux kernel and the SBC
   hardware. The SBC is your choice, and different SBCs provide different security guarantees.
3. The Secure Boot implementation on the SBC. Some don't provide secure boot at all, others have out-of-the box support
   for it. The concrete implementation and what security guarantees it provides is highly manufacturer-dependent.
4. The manufacturing and programming of the device before the tamper sensor is armed, as well as what happens to the
   erased device after an alarm or decommissioning.

Functional Components
~~~~~~~~~~~~~~~~~~~~~

The prototypical Ashen setup analyzed here consists of a SBC (think "raspberry pi" or similar) running an Ashen-equipped
kernel along the Ashen userspace components. The SBC is connected through UART to a Yanartas tamper sensor hardware
module running Ashen firmware. The tamper sensor runs the Ashen firmware. On the SBC, the Ashen Linux kernel driver is
running and communicates with the tamper sensor's firmware through the heartbeat protocol.

The tamper sensor is configured such that its sensing volume covers both the SBC and the tamper sensor itself. In this
document, we refer to the tamper sensor's sensing volume as the *protected volume*.

Lifecycle Model
~~~~~~~~~~~~~~~

In this analysis, we consider a three-phase lifecycle model. The phases are:

1. Commissioning Phase (out of scope for this analysis, refer to the `Commissioning Guide`_)
2. Operating Phase (scope of this analysis)
3. End of Life Phase (out of scope for this analysis)

The commissioning phase encompasses everything that happens before the Yanartas tamper sensor is armed. It includes
sourcing parts, manufacturing the device, programming it, and arming the tamper sensor. The operating phase starts once
the tamper sensor is armed, and ends either with the alarm being triggered by a tampering attempt, or by a
decommissioning command. In both cases, the system transitions to the End of Life phase by erasing its
`Critical Assets`_.

While the security model of an Ashen/Yanartas system is similar to that of a conventional HSM once it's deployed, its
commissioning phase is noticeably different. In a conventional HSM, the HSM manufacturer assumes all responsibility for
the proper manufacturing of the HSM according to its specifications, for sourcing components ensuring no supply chain
attacks happen, and programming the device. The tamper sensors of a conventional HSM are armed at the end of the
manufacturing process before the device is shipped out from the manufacturer. After purchasing such an HSM, the customer
can check that it wasn't tampered with in the meantime using an authenticity certificate programmed into the device by
the manufacturer.

In an Ashen setup, the assumption is that the user is assembling and programming the system themselves, and thus they
take on those responsibilities that would be the manufacturer's in a conventional HSM.

Security Model
--------------

Assumptions
~~~~~~~~~~~

Secure Device Commissioning.
    The Ashen user will assemble, program, configure and arm the device such that it meets all technical specifications,
    and that at the time of arming there are no pre-existing flaws such as compromised hardware components or malicious
    code. See the Commissioning Guide for guidance on achieving thsi in practice.

Yanartas Tamper Sensors Work.
    The scope of this analysis is the Ashen software stack and the interplay between its Linux and Sensor halves. We
    assume that the tamper sensor detects tampering. If tampering remains undetected, in general the security of the
    system will quickly degrade to that of a system without Ashen (but not worse).

Payload Mechanical Robustness.
    This analysis assumes the user mechanically constructs the HSM in a way that the attacker cannot destroy a small
    part of the payload SBC in a targeted way. One way to achieve this is if the payload SBC is enclosed in a robust
    (e.g. solid steel) enclosure that is inside the protected volume. The attacker is allowed to destroy the payload,
    just not in a targeted way and without tripping the tamper sensor.

Critical Assets
~~~~~~~~~~~~~~~

The following items are the key assets to be protected by the setup. Other data, most notably encrypted user data stored
in on-board flash on the SBC or in external storage is not included here since it is useless without the full-disk
encryption keys listed below.

Confidentiality
***************

The following items must be protected from being disclosed to the attacker.

* Heartbeat connection keys in the Yanartas sensor: Long-term an ephemeral asymmetric keys and the ephemeral symmetric encryption key
* Corresponing to the above, heartbeat connection keys on the running SBC
* Any user keys stored in the Yanartas sensor
* Full Disk Encryption (FDE) working keys located in CPU registers and main memory on the running SBC
* Plaintext user data sitting in main memory caches on the running SBC

Integrity
*********

The following items can safely be disclosed to the attacker, but must be protected against modifications by the
attacker.

* The tamper sensor firmware
* The SBC Software
* The communication link between tamper sensor and Linux kernel module

Availability
************

The following items must be available at all times, and them becoming unavailable should be considered equal to an
attack.

* The communication link between tamper sensor and Linux kernel module
* The tamper sensor firmware must be running at all times so it can respond to tampering attempts.

Data Flows and Trust Boundaries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Attacker Model
--------------

Attacker Capabilities
~~~~~~~~~~~~~~~~~~~~~

The attacker is assumed to be able to do the following things.

0. The attacker has access to the full source code, configuration (excluding secrets), and schematics and other CAD data
   of the system.
1. The attacker can observe the system from the outside in any region of the electromagnetic (EM) spectrum.
2. The attacker can perform X-ray and CT imaging of the system.
3. The attacker can analyze radiated electromagnetic emissions of the system across any surface outside the volume
   protected by the tamper sensor.
4. The attacker can analyze conducted EM on any cable entering the protected volume at the volume boundary.
5. The attacker can perform electromagnetic fault injection attacks against the tamper sensor.
6. The attacker can perform an on-path attack with the capability to read, delay, block, or modify UART transmissions in
   either direction.
7. The attacker can apply state-of-the-art forensic techniques such as those from semiconductor fault analysis to any
   component of the system in the End of Life phase.
8. The attacker can interrupt any internal electrical connection inside the protected volume using a contactless laser
   attack. 
8. The attacker can create new electrical connections between adjacent conductors on circuit boards inside the protected
   volume using a contactless laser attack. 
9. The attacker can selectively destroy the tamper sensor without destroying the SBC payload.

Attacker Types
~~~~~~~~~~~~~~

Attack Trees
------------

.. _`Commissioning Guide`: secure-commissioning-guide.rst
