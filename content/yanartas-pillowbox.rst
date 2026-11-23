---
title: "The Yanartas Pillowbox Generator"
weight: 80
---

The Yanartas Pillowbox Generator
================================

A pillow box is a type of folding cardboard package made from only six panels that is often used for small items.

This repository contains a generator tool, `yanartas-pillowbox`, that creates patterns for tamper-sensing meshes laid
out as foldable pillow boxes made from flexible printed circuits (FPCs). Compared with other ways of wrapping a flexible
mesh around a payload, a pillow box consists of fewer panels, and has better protection against ingress at its corners
and edges.

.. note:: 

   This code is entirely vibe-coded. Care was taken to make sure the outputs are reasonable, but the code constitutes a
   significant cognitohazard. Perhaps avoid interacting with this code directly, and carefully review its output for
   correctness. Since this is a one-off tool that is only used during the design phase, we consider this an acceptable
   tradeoff for a (much) faster development timeline and less human life spent on debugging computer geometry code.

Pillowbox Layouts
-----------------

The tool can create two types of pillowbox layouts, a regular pillowbox layout and a split layout. The regular layout is
a single, symmetrical piece that you fold up in the middle before gluing it along one of the long body edges. The split
layout is similar, but instead of folding the two body panels from one piece, it uses two copies of a smaller piece that
are glued together along two edges to form the complete body.

The split layout has one more edge that needs to be glued, but saves about a factor of two in board size and area.

Mesh Layout Algorithms
----------------------

The tool integrates two algorithms to layout the mesh traces inside the folding patterns' panels. First, it ships with a
port of the classic kimesh_ algorithm. This algorithm produces meshes of a well-defined, near constant density that
guarantee no layout defects, but due to the pixelated structure of these meshes they do not conform to edges of the
design well and they do not take fold lines between panels into account.

Second, the tool integrates an advanced iterative layout algorithm that conforms nicely to edges and fold lines. We
recommend this algorithm for all meshes. 
The layout algorithm is based on research by Adrien Bedel, Yoann Coudert-Osmont, Jonàs Martínez, Rahnuma Islam Nishat,
Sue Whitesides, et al. that can be found `here <inria-paper_>`_ with a reference implementation available at the
`institute's github <inria-source_>`_.

FPC Layout
----------

`yanartas-pillowbox` includes code that automatically lays out an FPC cable attached to one of the long body edges of
the mesh to connect the mesh to a tamper monitoring circuit. See the `yanartas overview page <./yanartas/>`__ for
details on these circuits.

.. _kimesh: https://jaseg.de/projects/kimesh/
.. _inria-paper: https://inria.hal.science/hal-03185200/document?utm_source=consensus
.. _inria-source: https://github.com/mfx-inria/controllable-space-filling-curve
