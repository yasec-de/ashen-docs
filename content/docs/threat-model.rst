---
title: "Threat model"
---

System Description
------------------

This page contains a semi-formal analysis of the security of a *typical* Ashen-based HSM. The HSM we describe here is a
generic setup based on a *typical* Linux Single-Board Computer (SBC) running Ashen on Linux that is connected to a Yanartas
tamper sensor. You can use this threat model as a starting point for tailoring a more comprehensive threat model
covering your particular use of Ashen.

Scope
~~~~~

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
5. Whatever application software the user runs on the Linux SBC is not in scope.

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

Linux Application Security.
    The user application running on the Linux SBC itself is out of scope. Thus, we consider anyting that happens through
    this software to be authorized. We assume the user application will never grant the attacker access to the SBC's
    Linux installation.

Critical Assets
~~~~~~~~~~~~~~~

The following items are the key assets to be protected by the setup. Other data, most notably encrypted user data stored
in on-board flash on the SBC or in external storage is not included here since it is useless without the full-disk
encryption keys listed below.

Confidentiality
***************

The following items must be protected from being disclosed to the attacker. A succcessful attack is one where the
attacker gains at least partial knowledge of one of these secrets.

* Heartbeat connection keys in the Yanartas sensor: Long-term an ephemeral asymmetric keys and the ephemeral symmetric encryption key
* Corresponing to the above, heartbeat connection keys on the running SBC
* Any user keys stored in the Yanartas sensor
* Full Disk Encryption (FDE) working keys located in CPU registers and main memory on the running SBC
* Plaintext user data sitting in main memory caches on the running SBC. Note that here attack success only encompasses
  access outside of what is otherwise public knowledge or ordinarily available through the user application.

Integrity
*********

The following items can safely be disclosed to the attacker, but must be protected against modifications by the
attacker. Attack success is defined by the attacker at least changing the target information.

* The tamper sensor firmware.
* The SBC Software.
* The plaintext payload of the communication link between tamper sensor and Linux kernel module.

Availability
************

The following items must be available at all times, and them becoming unavailable should be considered equal to an
attack. Attack success is defined as the attacker compromising availability undetected, i.e without triggering a tampre
alarm, for a period greater than the system's heartbeat protocol timeout.

{{< hint warning >}}
    As a concept of this model, the attacker <emph>is allowed</emph> to temporarily disturb availability for a period up
    to the system's heartbeat protocol timeout, which is typically in the dozens of milliseconds. The practical
    consequences of this are minimal, as this time is deliberately kept short to prevent the attacker from mounting a
    successful physical attack in the meantime. Furthermore, for typical networked applications, the system's heartbeat
    protocol timeout will likely be in the same order of magnitude as unavoidable network latency. However, this timeout
    constitutes what is essentially a metastable period in the system's integrity state. In certain highly
    time-sensitive applications such as fast blockchain or other distributed systems, it may be useful to delay some
    actions until a confirmation period greater than this heartbeat timeout has passed.
{{< /hint >}}

These items are covered by our availability requirement:

* The communication link between tamper sensor and Linux kernel module must be available.
* The tamper sensor firmware must be running at all times so it can respond to tampering attempts.
* We do not consider intentional triggering of the tamper alarm by an attacker with physical access to the device a
  successful attack. However, we consider it an attack if the attacker is able to trigger the alarm *without* physical
  access.

Data Flows and Trust Boundaries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The setup described above contains two primary trust boundaries. The exterior boundary lies between the application
software on the Linux SBC and the outside world. The second, interior trust boundary lies between this application
software and the tamper sensor. The interior trust boundary is necessary since it allows the Linux SBC to power down
without compromising the system's security by offloading its root key material to the tamper sensor while the SBC is
powered off.

Attacker Model
--------------

Attacker Capabilities
~~~~~~~~~~~~~~~~~~~~~

The attacker is assumed to be able to do the following things, possibly several of them simultaneously.

Knowledge and Access
********************

1. The attacker has access to the full source code, configuration (excluding secrets), and schematics and other CAD data
   of the system. The attacker furthermore has access to identical, devices (though without the target's key material).
2. The attacker can physically access the system, but cannot mechanically enter the protected volume using tools without
   tripping the tamper sensor.
3. The attacker can acccess the system for extended periods of time (up to several months).
4. The attacker can move the system for small distances up to a kilometer undetected, but cannot move it across larger
   geographical distances undetected.
5. The attacker has full network access to any interfaces exposed over cables entering the protected envelope.

Passive Observation
*******************

All of these attack methods require physical access to the device.

1. The attacker can observe the system from the outside in any region of the electromagnetic (EM) spectrum. This
   includes RF, but also optical, and thermal infrared emissions.
2. The attacker can observe acoustic emissions from the system.
3. The attacker can perform X-ray and CT imaging of the system.
4. The attacker can analyze radiated electromagnetic emissions of the system across any surface outside the volume
   protected by the tamper sensor.
5. The attacker can perform conducted EM and power side channel analysis on any cable entering the protected volume at
   the volume boundary.

Active Manipulation
*******************

All of these attack methods require physical access to the device.

1. The attacker has full access to all power and data connections entering the protected envelope, and can disconnect
   them.
2. The attacker can manipulate the environmental conditions of the system. This includes temperature, vibration,
   ionizing radiation, magnetic fields.
3. The attacker can perform Electromagnetic Fault Injection (EMFI) attacks against the tamper sensor. They cannot
   perform EMFI against the payload.
4. The attacker can perform an on-path attack with the capability to read, delay, block, or modify UART transmissions
   in either direction.
5. The attacker can apply state-of-the-art forensic techniques such as those from semiconductor fault analysis to any
   component of the system in the End of Life phase.
6. The attacker can interrupt any internal electrical connections inside the protected volume using a contactless laser
   attack. 
7. The attacker can create new electrical connections between adjacent conductors on circuit boards inside the
   protected volume using a contactless laser attack. 
8. The attacker can selectively destroy the tamper sensor without destroying the SBC payload.

Attacker Motivation
~~~~~~~~~~~~~~~~~~~

Comparable threat modeling documents for other systems sometimes adopt schemata of attacker types, assigning imaginary
capabilities to each attacker type describing what they can do and what they cannot. We omit this sort of categorization
of attacks since it's not very useful. Such taxonomies often result in realistically untenable assumptions like that
nation-state adversaries cannot have their spies apply for jobs at a target organization, or that hacker catgirls do not
have access to focused ion beam rework stations.

Instead of this taxonomical approach, we propose some scenarios illustrating why an attacker may be willing to expend
resources ot attach an Ashen-based HSM. Common to these scenarios is that the HSM may be located in a data center or
other facility outside of its operator's immediate physical access, and as such attackers may have physical access to
the device for extended periods of time.

Scenario 1: Activist Video Conference Server
********************************************

In this scenario, an activist group operating in a hostile state runs a standard video conference software on the
HSM-protected Linux SBC. The protection goal of the system is to prevent corrupt police forces from compromising the
video conference system and gaining access to communication contents and participant metadata. 

Scenario 2: Device Factory Certificate Signing Provider
*******************************************************

In this scenario, a hardware designer provisions a HSM-protected Linux SBC inside of a third-party factory. The system
contains an asymmetric private key that application software running on it uses to provide device authenticity
certificates for devices manufactured in the factory. The system furthermore creates an authenticated audit log of all
issued certificates that the hardware designer can use to assure compliance of the factory with production quota.
Attackers in this scenario include malicious third parties attempting to compromise the manufactured devices. The range
of relevant attackers further includes the operators of the factory, who may be incentivized to run a third shift,
producing excess devices and selling them through illicit channels without the hardware designer's knowledge.

Scenario 3: Software Release Signing Server
*******************************************

In this scenario, a volunteer-run Linux distribution builds binary software packages for their package repositories. At
the end of the build process, the binary packages are signed to attest to their authenticity when users download them
from a swarm of mirror servers. The volunteer group uses some custom-built software running on a HSM-protected Linux SBC
to generate these signatures while also keeping a tamper-proof audit log. Attackers in this scenario would be
cybercriminals or nation-state adversaries attempting to compromise build artifacts.

Attack Success
~~~~~~~~~~~~~~

We define a successful attack as one where the attacker manages to compromise one of the `Critical Assets`_ defined
above. Depending on the asset, that means for instance disclosure of a secret or denial of service.

{{< hint info >}}
We consider triggering of the tamper alarm a failed attack. We do not consider the trivial denial of service of an
attacker intentionally triggering the tamper alarm a successful attack if the attacker has physical access to the
system. However, if the attacker can trigger the tamper alarm remotely *without* having had phsical access at any point
in time, we do consider that a successful attack.
{{< /hint >}}

Attack Trees
------------

.. _`Commissioning Guide`: ../secure-commissioning-guide
