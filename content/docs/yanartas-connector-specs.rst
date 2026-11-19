---
title: "Yanartas Mesh Connector Specification"
---

The Yanartas Mesh Connector Specification
=========================================

To allow Yanartas designs to share meshes between sensor board variants, we're using the following standardized
interface between mesh and sensor board.

Yanartas 30-Pin Mesh Connector
------------------------------

The mesh connector is a standard 30-pin, 0.5mm pitch FPC connector. The sensor board contains the receptacle. The mesh
can either integrate the mating connector in case of a FPC mesh, or use a second receptacle and a 30 pin FFC cable.

The reference mesh connector type on the sensor board is `Hirose FH12S-30S-0.5SH`_. The dimensions of the mating
FPC connector can be taken from Hirose's datasheets. Common 0.5mm FFC cables work.

The pinout of the connector is as follows:

FIXME

.. _`Hirose FH12S-30S-0.5SH`: https://www.hirose.com/en/product/p/CL0586-0525-1-55
