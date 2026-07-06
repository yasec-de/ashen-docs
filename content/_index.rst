---
title: "The Ashen Open Source HSM Stack"
---

What is Ashen?
==============

In the Ashen project, we create a fully open source software and hardware stack for building open source Hardware
Security Modules (HSMs). A HSM is a computer that is protected against advanced hardware attacks such as probing_ 
and `fault injection`_ attacks. Up to now, HSMs are only used in niche applications because they are expensive to buy
and difficult to program. The goal of Ashen is to make it so that anyone with a basic technical understanding can build
their own HSM and protect their data from advanced attacks.

How do HSMs work
----------------

We wrote above that a HSM "protects" a computer from hardware attacks. At first, you might think that you can just put
the whole computer inside of a safe. However, in practice this doesn't really work because safes are in fact not very
safe when the attacks has time and tools. As the US NSA's David G. Boak put it in his
`1981 lecture on US communications security`_:

    One of our most intractable problems has been to find ways to package crypto-equipment in a way which will seriously
    deter penetration by a smart, well-equipped opponent with plenty of time. The difficulty is not much different than
    is faced in the manufacture of three-combination safes. The best we can generally afford can stand up to a covert
    penetration effort by an expert only for 30 minutes or so. and brute force attacks, leaving evidence, can be done
    much more quickly than that. Yet, these safes are massive and expensive. With a crypto-box, there are added
    difficulties in protecting logic or resident key because X-ray devices or electronic probing may recover the
    information without physical entry.

Thus, HSMs do not try to physically stop an attacker attempting to open them. Instead, they are built such that any
attempt to open them will trigger an alarm that destroys all secrets stored inside the HSM. Attackers can easily open a
HSM with little more than a hacksaw, but doing so becomes unattractive because by the time they are done, anything
interesting that was once inside the device will have vanished.

What does Ashen do?
-------------------

While the concept of a HSM sounds simple, getting one to work reliably in practice is hard. Ashen is meant to solve
exactly this hard problem, to make it easy to build a HSM. Ashen will:

What is Ashen made from?
========================

An Ashen HSM is made up from the following parts:

.. _`1981 lecture on US communications security`
