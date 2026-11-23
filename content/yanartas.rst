---
title: "The Yanartas Tamper Sensor"
weight: 15
---

What is Yanartas?
=================

In the Yanartas project, we are creating a family of tamper sensor hardware designs that are designed to run
ashen-firmware_. The first sensor we are creating is an advanced mesh-based tamper sensor using `Time Domain
Reflectometry (TDR) <tdr_>`_, with later designs adding environmental sensors and other niceties.

.. important::

   Ashen is currently work in progress! It's currently at the design stage, with prototypes planned for end of the year
   2026. jaseg_ is working on it full time. Prototypes are planned for early December, and we are planning a first
   release for late spring 2027. We're open for collaborations and contributions—feel free to reach out via
   `email <ashen@yasec.de>`__.

Tamper Sensors
==============

The goal of a HSM tamper sensor is to detect someone trying to physically manipulate the HSM, without triggering false
alarms during normal operation. Manipulations someone could attempt range from simply looking at the device, through
probing electrical signals using e.g. an oscilloscope, up to attacks like fault injection or micro-probing.

The usual way to implement HSM tamper sensors is to package the HSM in some reasonably sturdy, shielded enclosure, often
using potting. The tamper sensors themselves are then usually monitoring the integrity of the whole, potted enclosure
more than they monitor the actual HSM circuitry that they're protecting. Common tamper sensors fall in the following
three categories.

Environmental Sensors
---------------------

The most basic sensors monitor the HSM's environmental conditions for anything unusual for normal operation. As a
minimum, this means measuring temperature and raising an alarm when it moves out of normal operational bounds (e.g. 0 -
60°C). Temperature sensors can preempt some physical attacks such as `Memory Remanence Attacks`_ that either cool
the device to preserve memory state across a reset, or heat it to cause accelerated aging of memory cells.

On top of temperature sensors, voltage sensors on power rails as well as clock frequency watchdogs on the system's clock
sources serve as a simple system health check and can detect some types of attacks such as certain voltage fault
injection attacks that require lowering the system voltage below safe boundaries.

Ionising radiation sensors, which in the simplest case can be a single large-area photodiode, can detect attempts to
image the interior of the HSM using X-ray or CT imaging.

Visible light and gas pressure sensors can be used to make it more difficult for an attacker to open a sealed enclosure.

Mechanical sensors
------------------

Monitoring the basic mechanical integrity of the HSM can also be quite effective. Simple measures include switches that
detect the removal of enclosure components, electrical contacts at screws that detect removal of the screw, and
pressure-sensitive contacts in stacks of enclosure parts that detect disassembly. Beyond HSMs, in the wild several types
of these contacts are commonly found in card payment terminals, where they are used to defend against criminals
trying to install skimmers_.

Tamper Sensing Meshes
---------------------

While environmental monitoring and mechanical integrity sensors are already quite effective, it's conceivable that a
skilled attacker could circumvent them, e.g. by bridging mechanical contacts or drilling out photodiodes in the dark
before disassembly. To detect against such advanced attacker, a Tamper Sensing Mesh (TSM) can be used as a second line
of defense. A TSM is nothing more than a circuit trace embedded in the HSM that is designed to cover as much area as
possible with as little gaps as possible such that an attacker has a hard time cutting or drilling through the TSM to
access whatever lies behind.

In HSMs, TSMs are commonly implemented as a `Flexible Printed Circuit (FPC)`_ on a foil
that is wrapped around the HSM's protected payload. Key criteria for the design of a good tamper-sensing mesh are that
it covers its payload without gaps (especially at corners), and that it is difficult to remove intact. To achieve the
latter, TSMs are often made from fragile materials and then potted inside a strong resin.

If you want to have a closer look at how TSMs are done in practice, have a look at `this paper <survey-paper_>`_ in
which jaseg_ present the results of a large scale survey of TSMs found in the real world.

TDR-Based Mesh Monitoring
-------------------------

Environmental sensors are easy to include in a HSM design. All you need to do is to drop a ready-made sensor such as a
photodiode or a temperature sensor into your design. In contrast, *effective* Tamper-Sensing Meshes are much harder to
design. In commercial HSMs, the mesh foils are expensive components made with bespoke, special-purpose machines. One
goal of the Yanartas project is to make effective TSMs accessible to hackers without all of this bespoke equipment using
open source. We achieve this by combining a low-tech TSM made from a normal circuit board that wouldn't be very secure
on its own with a high-tech sensing approach based on `Time Domain Reflectometry <tdr_>`_. The details of the sensing approach
are laid out in a `paper <tdr-paper_>`_ published by project lead jaseg_ during their PhD.

The Yanartas Designs
====================

`unoq-baseline: Yanartas Unoq Demo Mesh Adapter <unoq-baseline-repo_>`_
-----------------------------------------------------------------------

`unoq-baseline` is a really simple design that allows connecting two Tamper-Sensing Meshes or other tamper sensors to an
Arduino Uno Q Linux Single-Board Computer. The Uno Q already contains a powerful STM32U585_ microcontroller that is
easily able to run ashen-firmware_. This board is a simple adapter between the Uno Q and the ``FFC-30P-6M4`` `Yanartas
30-pin mesh connector <../docs/yanartas-connector-specs>`_.

**Status:** In development, first prototype expected by end of November 2026.

`proto-01-transparent: See-through Pillowbox Mesh Demo <https://git.yasec.de/yasec/yanartas-pillowbox-protos/src/branch/main/proto-01-transparent>`__
------------------------------------------------------

`proto-01-transparent` is a see-through pillow box tamper sensing mesh made using the `Yanartas Pillowbox Generator`_.
It's mostly for demonstration use, because you can see through it. It lacks the ground planes that would be necessary to
use it with the TDR sensing board, but it can be used in continuity/short sensing mode with the unoq demo board. The
board uses the first four out of the six mesh segments on the the the ``FFC-30P-6M4`` `Yanartas 30-pin mesh connector
<../docs/yanartas-connector-specs>`_.

**Status:** Prototypes finished and confirmed working.

`warp: TDR Prototype <warp-repo_>`_
-----------------------------------

`warp` is a TDR-only sensor board prototype with a carrier board designed to take a Raspberry Pi Compute Module 5.
`warp` does not include any other tamper sensors and is meant for development of the TDR sensing frontend and firmware.
The board supports the full pinout of the ``FFC-30P-6M4`` `Yanartas 30-pin mesh connector
<../docs/yanartas-connector-specs>`_ and carries a powerful STM32 microcontroller capable of running the full
ashen-firmware stack.

**Status:** In development, first prototype (version ``v1 2026-11-16``) expected by end of November 2026.

weft: Environmental sensor prototype
------------------------------------

Complementing the `warp` TDR mesh sensor, `weft` is a prototype design exploring various environmental sensors. `warp`
is based on the Arduino Uno Q platform and is derived from `unoq-baseline`. It still carries the baseline's
continuity/short only mesh sensing connectors, which are simply passed through to the STM32 like in the baseline design.

**Status:** In development at the system design stage. First prototypes are expected end of December 2026.

.. _tdr: https://en.wikipedia.org/wiki/Time-domain_reflectometer
.. _`Memory Remanence Attacks`: https://en.wikipedia.org/wiki/Cold_boot_attack
.. _skimmers: https://en.wikipedia.org/wiki/Credit_card_fraud#Skimming
.. _`Flexible Printed Circuit (FPC)`: https://en.wikipedia.org/wiki/Flexible_electronics#Manufacturing
.. _jaseg: https://jaseg.de/
.. _tdr-paper: https://tches.iacr.org/index.php/TCHES/article/view/12692
.. _survey-paper: https://eprint.iacr.org/2026/2475
.. _STM32U585: https://www.st.com/en/microcontrollers-microprocessors/stm32u575-585.html
.. _conn-spec-30-pin: ./docs/connector-specs
.. _ashen-linux: https://git.yasec.de/yasec/ashen-linux
.. _ashen-firmware: https://git.yasec.de/yasec/ashen-firmware
.. _ashen-library: https://git.yasec.de/yasec/ashen-library
.. _`Yanartas Pillowbox Generator`: https://git.yasec.de/yasec/yanartas-pillowbox-generator
.. _warp-repo: https://git.yasec.de/yasec/yanartas-warp
.. _unoq-baseline-repo: https://git.yasec.de/yasec/yanartas-warp
