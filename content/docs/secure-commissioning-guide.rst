---
title: "Secure Commissioning Guide"
---

In Ashen's security model, the *commissioning* of the system, i.e. acquiring, assembling, programming, and configuring
its components, is explicitly out of scope and up to you, the user. This guide will help you get to a secure Ashen
setup. It highlights what you need to keep in mind, and which steps are the most likely to result in a security problem.

{{< hint info >}}
    There is no absolute security, especially in hardware. A sufficiently determined and well-funded attacker will be
    able to overpower any defence you mount, even if just by sheer luck, tenancy, or brute force. However, by
    implementing all of these suggestions, you can make it a lot more annoying and much more expensive for an attacker
    to achieve their goals, and thus discourage attacks before they happen.
{{< /hint >}}

Executive Summary
-----------------

* Make sure your IT infrastructure is airtight.
* Keep your supply chains short and trustworthy. Avoid sketchy stops in your component's journey to you.
* Verify hashes and signatures of software downloads. 
* Do the final assembly and programming in house.

Hardware Assembly
-----------------

Component Procurement
~~~~~~~~~~~~~~~~~~~~~

Depending on the kinds of adversaries you are facing, you may need to consider supply-chain attacks. There's no one way
to fully prevent them, but there are some things you can do to make them much more difficult to implement in practice. A
basic assumption we make here is that unless we all are exceptionally unlucky, an attacker likely will not have a
backdoor pre-installed into all components of a type such as SBC or microcontroller. Instead, we estimate it's
reasonable to assume that the most likely supply-chain attack will be *targeted*, with the attacker modifying one or a
few components on the way to the victim.

* Make sure your vendors are trustworthy. Ideally, ask around for recommendations or stick to well-known component
  vendors.
* Check incoming components and their packaging against previous shipments. Verify part markings match the datasheet.
  Check that the packaging is complete and that its inner layers are undamaged. Check that identical parts weigh the
  same.
* Prefer sourcing critical components like SoMs or complex ICs like microcontrollers in ways that are harder to predict,
  and thus harder to intercept for attackers. A really solid way of achieving this is to physically go to a store and
  buy the necessary components. Where that's not possible, consider obscuring the final destination of the components
  when online shopping as long as that doesn't put you into trouble with export compliance laws.
* Try to avoid problematic jurisdictions ending up on the transit route of your packages.
* Keep your supply chains short and be wary of good deals.

Assembly
~~~~~~~~

Programming
~~~~~~~~~~~



Initial Trust Establishment
~~~~~~~~~~~~~~~~~~~~~~~~~~~

After the system has been build and programmed, it has to be booted once to generate keys and establish trust. Here is
what happens during this first boot:

1. Both the Linux SBC and the tamper sensor generate their long-term heartbeat protocol identity keys.
2. The Linux SBC generates keys for full-disk encryption and formats its storage with them.
3. The Linux SBC and the tamper sensor complete a first heartbeat protocol handshake, and store each other's long-term
   identity keys.
4. The Linux SBC prints the static handshake protocol identity key fingerprints of both itself and of the tamper sensor
   to the provisioner. They can be provided out of band to remote clients to enable the tamper sensor to remotely attest
   to the device's security after field provisioning.

After trust establishment, the SBC can be powered off, but the tamper sensor must remain continuously powered for the
remainder of the HSM's operating life. In particular, this means that the tamper sensor must remain powered while the
HSM is transported to its field operating location.

Field Provisioning
------------------

After assembly and trust establishment, the HSM is fully protected and can be installed at its final, untrusted
operating location. Since at this point, the tamper sensor is armed, no particular care has to be taken regarding the
device's security during transport or installation. After installation, users can ascertain the HSM's identity using the
two identity keys printed during the trust establishment step.

Maintenance
-----------

If at some point, the HSM needs to be disassembled for maintenance or repair, the process is similar to initial
provisioning. First, the clients must be alerted of the impending maintenance and all data stored on the HSM needs to be
backed up by them. Then, the HSM can be transported to a secure location while the tamper sensor is still armed. At the
secure location, the identity of the HSM must be confirmed through its two identity keys. When everything is alright, it
can simply be disassembled, intentionally triggering a wipe. After repair is complete, the HSM is re-programmed as if it
was newly built, and the trust establishment step is run again. After trust establishment, the new keys of the HSM are
provided to its clients out of band. After the HSM is installed in the field again, the clients can ascertain its new
identity and restore it from backups.

