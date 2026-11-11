---
title: "Ashen Firmware Architecture"
---

An Ashen-equipped system consists of a *host*, usually some sort of Linux computer, that is connected to a tamper sensor
module runing the Ashen firmware. The role of the Ashen firmware is to monitor tamper sensors, and to provide a
continuous, cryptographically secured proof of liveliness to the host system.

The current reference platform for Ashen is an Arduino Uno Q development board. On this board sits a large QRB2210 ARM
System on Chip that is running linux, which acts as the Ashen host. Also on the same board is a smaller STM32
microcontroller that runs the Ashen firwmare. For development, on this board the Ashen firmware uses a single GPIO pin
for tamper monitoring. In the Yanartas_ project, we will develop dedicated Ashen sensor modules that can be connected to
any Linux-capable host with an USB or UART port and that will have more advanced tamper sensors.

Key Management using Ashen
--------------------------

In a basic setup, the host talks to the Ashen firmware through its Heartbeat interface and uses the Ashen Linux kernel
extension to quickly wipe all secret date when a tamper alarm happens. In this setup, the host generally can't shut
down or reboot, as during that time it would be defenseless against an attacker. Even if the Ashen firmware detected an
attack, it could do nothing to prevent the attacker from simply copying the host's storage.

If we want to let the host reboot or power off for a while, its secrets must be kept inside the Ashen firmware so that
it can delete them if it detects an attack while the host is down. To avoid having to store every single secret from TLS
keys to user passwords in the Ashen firmware, the Ashen firmware acts towards the host like the Trusted Platform Module
in any modern computer. To achieve this, the Ashen firmware embeds WolfTPM_, a TPM 2.0 implementation made by the same
people that also make the WolfSSL_ TLS stack. The firmware TPM emulated by Ashen stores its keys along with Ashen's own
secrets to make sure that they will be deleted alongside Ashen's secrets during a tamper alarm. The host communicates to
the embedded firmware TPM through standard TPM commands that are tunneled alongside Ashen's heartbeat protocol through
the same, cryptographically secure link to the host.

The nice thing about Ashen embedding a full firmware TPM is that with just a bit of driver code, you can use all the
infrastructure that exists around TPMs with ashen. A reasonable baseline setup that guarantees protection even when the
host is powered off or rebooting has the host encrypt its storage using Full Disk Encryption with keys provided by
Ashen's embedded TPM. 

