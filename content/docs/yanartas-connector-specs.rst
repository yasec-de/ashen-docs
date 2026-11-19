---
title: "Yanartas Mesh Connector Specification"
---

The Yanartas Mesh Connector Specification
=========================================

To allow Yanartas designs to share meshes between sensor board variants, we're using the following standardized
interface between mesh and sensor board.

Yanartas 30-Pin Mesh Connector FFC-30P-6M4
------------------------------------------

``FFC-30P-6M4`` is a mesh connector carrying 6 mesh segments of one diffpair each through a standard 30 pin, 0.5mm pitch
FPC connector. The sensor board contains the receptacle. The mesh can either integrate the mating connector in case of a
FPC mesh, or use a second receptacle and a 30 pin FFC cable.

The reference mesh connector type on the sensor board is `Hirose FH12S-30S-0.5SH`_. The dimensions of the mating
FPC connector can be taken from Hirose's datasheets. Common 0.5mm FFC cables work.

.. hint::

   Locking FPC connectors are available in two variants, either with the contacts on the top side or with contacts on
   the bottom side of the FPC. Both variants will have mirrored pinouts. The lever-style locking FPC connectors like the
   FH12S series usually have their pins on the bottom, and all pinout information here is based assuming pins on the
   bottom. Both connector variants can be used e.g.\ for mechanical reasons, but make sure to mirror the pinout when
   using a top-contact connector. 

The connector carries a total of six mesh loops, each with two traces that make up a differential pair. We are labelling
the loops ``L1...L6`` and the two traces in each loop `P/N` for positive/negative polarity within the differential pair.
So loop ``L1`` consists of traces ``L1P`` and ``L1N``, which leave the sensing board on connector pins
``SEND_L1P/SEND_L1N`` and return to it on pins ``RET_L1P/RET_L1N``.

Pinout table
~~~~~~~~~~~~

.. table:: Mesh connector pinout (Hirose FH12S-30S-0.5SH(55), 0.5 mm pitch, bottom contact)

   +-----+----------+-------+
   | Pin | Signal   | Trace |
   +=====+==========+=======+
   | 1   | GND      | —     |
   +-----+----------+-------+
   | 2   | SEND_L1N | L1N   |
   +-----+----------+-------+
   | 3   | SEND_L1P | L1P   |
   +-----+----------+-------+
   | 4   | RET_L1P  | L1P   |
   +-----+----------+-------+
   | 5   | RET_L1N  | L1N   |
   +-----+----------+-------+
   | 6   | GND      | —     |
   +-----+----------+-------+
   | 7   | SEND_L2N | L2N   |
   +-----+----------+-------+
   | 8   | SEND_L2P | L2P   |
   +-----+----------+-------+
   | 9   | RET_L2P  | L2P   |
   +-----+----------+-------+
   | 10  | RET_L2N  | L2N   |
   +-----+----------+-------+
   | 11  | GND      | —     |
   +-----+----------+-------+
   | 12  | SEND_L3N | L3N   |
   +-----+----------+-------+
   | 13  | SEND_L3P | L3P   |
   +-----+----------+-------+
   | 14  | RET_L3P  | L3P   |
   +-----+----------+-------+
   | 15  | RET_L3N  | L3N   |
   +-----+----------+-------+
   | 16  | GND      | —     |
   +-----+----------+-------+
   | 17  | SEND_L4N | L4N   |
   +-----+----------+-------+
   | 18  | SEND_L4P | L4P   |
   +-----+----------+-------+
   | 19  | RET_L4P  | L4P   |
   +-----+----------+-------+
   | 20  | RET_L4N  | L4N   |
   +-----+----------+-------+
   | 21  | GND      | —     |
   +-----+----------+-------+
   | 22  | SEND_L5N | L5N   |
   +-----+----------+-------+
   | 23  | SEND_L5P | L5P   |
   +-----+----------+-------+
   | 24  | RET_L5P  | L5P   |
   +-----+----------+-------+
   | 25  | RET_L5N  | L5N   |
   +-----+----------+-------+
   | 26  | GND      | —     |
   +-----+----------+-------+
   | 27  | SEND_L6N | L6N   |
   +-----+----------+-------+
   | 28  | SEND_L6P | L6P   |
   +-----+----------+-------+
   | 29  | RET_L6P  | L6P   |
   +-----+----------+-------+
   | 30  | RET_L6N  | L6N   |
   +-----+----------+-------+
   | MP  | GND      | —     |
   +-----+----------+-------+

.. _`Hirose FH12S-30S-0.5SH`: https://www.hirose.com/en/product/p/CL0586-0525-1-55
