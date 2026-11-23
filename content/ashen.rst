---
title: "The Ashen Open Source HSM Stack"
weight: 10
---

What is Ashen?
==============

In the work-in-progress Ashen project, we create a fully open source software and hardware stack for building open
source Hardware Security Modules (HSMs). A HSM is a computer that is protected against advanced hardware attacks such as
probing_ and `fault injection`_ attacks. Up to now, HSMs are only used in niche applications because they are expensive
to buy and difficult to program. The goal of Ashen is to make it so that anyone with a basic technical understanding can
build their own HSM and protect their data from advanced attacks.

Ashen is designed to run on commodity hardware with the help of a Yanartas_ tamper sensor.

Project state
-------------

.. important::

   Ashen is currently work in progress! It works, but it's at an alpha prototype stage. jaseg_ is working on it full
   time, and we are planning a first release for January 2027. We're open for collaborations and contributions—feel free
   to reach out via `email <mailto:ashen@yasec.de>`__.

How do HSMs work
----------------

We wrote above that a HSM "protects" a computer from hardware attacks. At first, you might think that you can just put
the whole computer inside of a safe. However, in practice this doesn't really work because safes are in fact not very
safe when the attacker has time and tools. As the US NSA's David G. Boak put it in his
`1981 lecture on US communications security`_:

    One of our most intractable problems has been to find ways to package crypto-equipment in a way which will seriously
    deter penetration by a smart, well-equipped opponent with plenty of time. The difficulty is not much different than
    is faced in the manufacture of three-combination safes. The best we can generally afford can stand up to a covert
    penetration effort by an expert only for 30 minutes or so, and brute force attacks, leaving evidence, can be done
    much more quickly than that. Yet, these safes are massive and expensive. With a crypto-box, there are added
    difficulties in protecting logic or resident key because X-ray devices or electronic probing may recover the
    information without physical entry.

Thus, HSMs do not try to physically stop an attacker attempting to open them. Instead, they are built such that any
attempt to open them will trigger an alarm that destroys all secrets stored inside the HSM. Attackers can easily open a
HSM with little more than a hacksaw, but doing so becomes unattractive because by the time they are done, anything
interesting that was once inside the device will have vanished.

What does Ashen do?
-------------------

While the concept of a HSM sounds simple, getting one to work reliably in practice is hard. Ashen is designed to make
this easy. Ashen is a set of software and hardware that you can combine with any Linux machine to make it into a HSM.
Ashen will:

* Monitor tamper sensors
* Alarm the machine when it detects an attack
* When an attack happens securely and quickly delete all secrets from the Linux machine
* Provide (optional!) programming interfaces to allow cryptography code running on Linux to label its secrets for quick
  deletion

What is Ashen made from?
========================

An Ashen HSM is made up from the following parts:

* The ashen-linux_ repository contains a Linux kernel driver that monitors tamper sensors, and that quickly deletes
  all memory when an attack happens.
* The ashen-library_  repository will contain libraries you can use to interact with ashen-linux_ to mark secret data
  in your application for prioritized deletion when an attack happens. ashen-linux will delete all memory on an attack
  either way, but for machines with lots of RAM that can take several seconds. Using ashen-library_ you can make sure
  that the most secret data, such as encryption keys, are deleted within milliseconds.
* The ashen-firmware_ repository contains the Ashen tamper sensor firmware. You can run this firmware on a
  microcontroller connected to a Linux computer to use it as a tamper sensor.
* In the Yanartas_ project, we are building purpose-made advanced tamper sensors that are designed to run
  ashen-firmware.
* The `Yanartas Pillowbox Generator`_ automatically creates flexible printed circuit (FPC) layouts for security meshes
  that fold up like a `Pillow Box`_. The project is listed here because it's not by itself hardware, but a pure software
  toolchain.

Project Roadmap
===============

All software components described above existin in working, prototype implementations. Currently, jaseg_ is working in
parallel on the first release of ashen, and on the first Yanartas_ tamper sensor reference hardware.

.. _`1981 lecture on US communications security`: https://archive.org/details/history_comsec_ii-nsa/history_comsec_ii-ISCAP
.. _jaseg: https://jaseg.de/
.. _`fault injection`: https://tches.iacr.org/index.php/TCHES/article/view/7390/6562
.. _probing: https://tches.iacr.org/index.php/TCHES/article/view/11802
.. _ashen-linux: https://git.yasec.de/yasec/ashen-linux
.. _ashen-firmware: https://git.yasec.de/yasec/ashen-firmware
.. _ashen-library: https://git.yasec.de/yasec/ashen-library
.. _Yanartas: ../yanartas
.. _`Pillow Box`: https://arxiv.org/html/2410.17593v1
.. _`Yanartas Pillowbox Generator`: https://git.yasec.de/yasec/yanartas-pillowbox-generator
