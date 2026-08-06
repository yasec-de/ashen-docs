---
title: "The Linux Wipe System"
---

When a HSM detects that an attacker is tampering with its hardware, it must respond by quickly wiping (deleting) all
secrets. The point of this is to discourage attackers from attempting to read out secrets using hardware attacks, or
else they will risk destroying all contents of the HSM. In conventional HSMs, wiping the secrets is usually done by just
removing the power supply of the memory that they are stored inside. In our Linux-based HSM, we can't do that and we
have to spend a little more effort.

Conventional HSMs use Static RAM, or SRAM, a type of memory that is very fast, but also very expensive and power-hungry.
SRAM can't easily be built larger than a few megabytes, while modern computers need gigabytes of memory. For this
reason, in modern computers, fast SRAM is only used for small *cache* memories inside the processor, and they use
slightly slower Dynamic RAM, or DRAM, for storing the rest of the memory contents that don't fit into these fast caches.
DRAM is what most people think of when you talk about computer memory, and can be made up to hundreds of gigabytes in
size.

The challenge in making a HSM out of a Linux computer comes in when you try to quickly delete this memory. While SRAM
can be erased reliably by simply disconnecting its power supply, DRAM cannot be erased as easily. While DRAM will not
store data permanently without power, it will retain its content for a short while--up to several minutes--when power is
lost. For this reason, a viable attack on DRAM is to unpower the computer, physically unplug the DRAM memory, and read
it out plugged into a different computer that the attacker controls.

For this reason, when an alarm happens, instead of disconnecting the power supply and relying on the DRAM memory content
to naturally decay, Ashen actively overwrites the DRAM content with meaningless garbage. The tricky part is that Ashen
has to do this overwriting both reliably, and very fast.

Fast DRAM Wipe on Linux
-----------------------

To speed up the wiping process, and to make sure the most secret data is erased almost instantly, Ashen splits the
memory wipe process into two stages:

1. Selectively erasing small secret memory areas for things like cryptographic keys from the Linux kernel
2. Stopping the Linux kernel and erasing the whole memory from start to end

We aim to make step 1 take less than 10 milliseconds, while step 2 takes about half a second to a second on an average
modern system. By staging the wipe process like that, we ensure that there is no opportunity for an attacker to
interrupt the wipe process after an alarm before at least all important secrets have been erased.

Selecting Secrets to Wipe
~~~~~~~~~~~~~~~~~~~~~~~~~

Ashen selects the targets to wipe in the first, high-priority stage by hooking into several standard Linux APIs. First,
any Linux kernel keyrings are erased. These keyrings include the full disk encryption keys for any unlocked encrypted
disks.

{{< hint warning >}}
    Currently, only software full disk encryption keys are wiped. Keys stored in special-purpose registers of inline
    disk encryption accelerator hardware are not currently supported.
{{< /hint >}}

After wiping the kernel keyrings, Ashen will proceed with erasing userspace memory that applications have explicitly
marked as containing secrets using the kernel's standard memfd_secret_ system. Applications don't currently mark secret
memory areas automatically, but it is reasonably easy to modify them to do this to improve their protection in an
Ashen-based HSM. As part of Ashen, we will create a userspace library that makes it easy to allocate such special secret
memory areas.

Halt and Catch Fire
~~~~~~~~~~~~~~~~~~~

`Halt and Catch Fire`_ is a programming meme referring to an processor instruction that locks up the processor core it's
running on until the system is restarted. Ashen, in effect is a practical implementation of this fictitious idea. In
Ashen, the wipe process is triggered by a `Kernel Panic`_ since that's a fast and realiable way to interrupt all other
processing activity on the system. We chose a kernel panic because when the system is under high load, a kernel panic
will take priority over anything else running on the system at the time.

When a kernel panic happens, the Linux Kernel will first stop all processor cores except for the one where the kernel
panic was started. Next, it will disable *preemption*. Preemption is the mechanism the kernel uses to interleave
multiple tasks on the same processor core. Once all cores except for the panicing core have been stopped, and after even
on that core, preemption has been disabled, there's nothing left that can interrupt the kernel while it's proceeding to
erase memory.

Since the kernel will never recover from the kernel panic anyway, the memory wipe process can erase memory without
needing to coordinate with whatever process or kernel thread that memory belongs to. Removing this coordination is
essential for speeding up the wipe process enough that it completes within the target of 10 milliseconds.

After the targeted wipes have been completed as part of the kernel panic process, Ashen lets the kernel panic continue
like it usually would. At the end of the panic process, Ashen uses a Linux feature named kdump_ to hand over execution
from the Linux kernel to a completely separate piece of software running outside of the kernel that will take over to do
the second pass of erasing all memory from start to end. This piece of software, called the *zeroize payload*, is kept
separate because the kdump mechanism is a clean way of getting code running with direct access to the full physical RAM
through a flat mapping.

{{< hint info >}}
    We wrote above that the wipe payload sees all of main memory through a flat mapping. There is one small caveat to
    this. Many ARM-based SBCs have a small operating system running in the ARM processor's TrustZone (TZ) facility.
    TrustZone is a technology for running isolated code next to an operating system on the same CPU. It is in essence a
    virtualization technology similar to conventional hypervisors.

    ARM System-on-Chip (SoC) makers as well as SBC designers often use TZ to handle hardware housekeeping tasks. In
    these cases, a small amount of firmware is continuously sharing the same CPU cores with the Linux host operating
    system. This firmware is designed to only minimally interact with the Linux host operating system, and partitions
    off its own memory areas both in SoC-internal memory and in the system's DRAM.

    This memory partitioning happens in the background, and is invisble to the Linux kernel. As a result, the view the
    Linux kernel has on main memory is technically incomplete, as there may be small memory areas that are used by TZ
    and that are invisble to the Linux kernel. In our design, this circumstance is of no consequence since these TZ
    memory areas should never contain user secrets.
{{< /hint >}}



.. _`Halt and Catch Fire`: https://en.wikipedia.org/wiki/Halt_and_Catch_Fire_(computing)
.. _memfd_secret: https://man7.org/linux/man-pages/man2/memfd_secret.2.html
.. _`Kernel Panic`: https://en.wikipedia.org/wiki/Kernel_panic
.. _kdump: https://www.kernel.org/doc/html/latest/admin-guide/kdump/kdump.html
.. _TrustZone: https://www.arm.com/technologies/trustzone-for-cortex-a
