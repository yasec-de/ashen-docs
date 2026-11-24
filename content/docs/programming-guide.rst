---
title: "Ashen Application Programming Guide"
---

This guide describes the high-level userspace APIs that you can use to integrate an application for fast memory wipe
with Ashen. By integrating an application with Ashen, you can make sure that when a tamper alarm is triggered, the
application's secrets will be wiped first, decreasing the window of opportunity for an attacker to extract its secrets.

Integrating an application with Ashen like this is a *hardening* measure. It can decrease secret exposure by erasing
secrets faster, but even without any hardening, on an alarm an Ashen-equipped system will delete all memory. Without
this integration ashen will just not know where in memory the secrets are, and it may take some time until the deletion
reaches the secrets. Depending on the system and its amount of memory, this can be hundreds of milliseconds up to a few
seconds.

Ashen provides APIs on three different levels of abstraction:

1. An application-level interface through podman/crun integration that can be used to run OCI containers in a microVM
   that has its memory tagged for early wiping.
2. An extension to the standard ``madvise()`` syscall that allows you to tag secret memory areas for early wiping from
   any systems programming language.
3. A standard TPM 2.0 interface that you can use to roll your own key management schemes that offload important
   cryptographic secrets to the ashen tamper sensing controller to keep them entirely out of Linux memory. 

Secret MicroVM-Based Containers
-------------------------------

The quickest way to tag an application's memory for early wipe is to run it as a container using podman's crun
integration.

Python Language Bindings
------------------------

Low-Level API
-------------

