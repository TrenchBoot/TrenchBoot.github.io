# Qualcomm DRTM

Following [assessment of DRTM on ARM](DRTM_On_ARM.md), this report covers an
examination of an ARM device featuring DRTM, a [ThinkPad X13s Gen 1][laptop]
laptop.  This device was released before ARM produced a DRTM
specification, but there isn't much public information about its DRTM
implementation, making it an interesting case to explore.

As will be made obvious below, the similarity with [DEN 0113][den0113] is
rather superficial, making this device unsuitable for porting TrenchBoot.
Still, tinkering with it provided some experience with AArch64, and
demonstrates the difficulties one might expect to face on other, more suitable,
devices.

[laptop]: https://psref.lenovo.com/product/ThinkPad/ThinkPad_X13s_Gen_1
[den0113]: https://support.arm.com/documentation/den0113/latest/
[secure-core]: https://learn.microsoft.com/en-us/windows/security/book/hardware-security-silicon-assisted-security#secured-core-pc-and-edge-secured-core

## Implementation overview

The laptop in question is one of ["Secured Core"][secure-core] Windows devices,
that rely on DRTM for their security features.  One may guess that this DRTM
implementation has been developed specifically for Windows-on-Arm, which has
important consequences:

- This DRTM implementation is unlikely to exist anywhere beyond this line-up of
  laptops.
- All such laptops should work about the same, unless the implementation has
  seen significant updates, which is not very likely.
- Another security feature of the laptops is that they come with firmware Secure
  Boot (not to be confused with UEFI Secure Boot) that doesn't allow running
  firmware unless the device's vendor has signed it.  This excludes the option
  of customizing firmware for implementing DRTM per DEN 0113 specification.

One of the first things to try was running [ACS DRTM][acs-drtm] tests, which
exercise the DRTM API from DEN 0113.  There is a somewhat outdated EFI binary
that can be easily executed from a UEFI Shell (although starting a shell
required a USB stick due to the BIOS lacking a "Boot from file" option).  While
the binary is from March 2025 and targets DRTM specification v1.1, the
results were unambiguous: the API per specification is not available.  This
could have been determined from the production date alone, but it's nice to be
able to confirm it directly.

Qualcomm's DRTM doesn't rely on hardware support beyond verifying BIOS at boot,
and in this sense is similar to a firmware-backed DRTM of DEN 0113.  In other
words, that firmware Secure Boot provides SRTM, which is used as the root of
trust in DRTM implementation (i.e., that the implementation itself can be
trusted).  In this sense, the implementation and everything else that runs at
EL3 is included in the TCB.

One good thing is that UEFI Secure Boot can be disabled, which is what allows
booting systems that aren't signed with Microsoft keys.

The overall structure of the implementation is broadly similar to DEN 0113:

- DRTM implementation runs at EL3
- the use of SMC calls for communication (a common thing on ARM)
- stateful API
- a parameter structure which is filled before the DRTM launch command is
  issued; it is used to specify the location of the TPM event log and what to
  execute after DRTM launch

If those similarities to the specification are counted as pros, here are the
cons:

- a vendor-locked implementation (only Microsoft-signed code is meant to be able
  to use the API)
- it's proprietary (no documentation, thus no easy way to reuse)
- more of an implementation detail than a generic solution (to be fair, it
  wasn't meant to be generic)
- not possible to replace (firmware Secure Boot prevents writing open-source
  firmware with a different DRTM implementation)

[acs-drtm]: https://github.com/ARM-software/sysarch-acs#drtm-architecture-compliance-suite

## slbounce

[slbounce][slbounce] is a project that exploits an error handling path of
`tcblaunch.exe`, which is a Microsoft-provided component of DRTM setup on these
laptops.  This project provides a way to utilize DRTM for something other than
Windows (e.g., Linux), mainly for the purpose of supporting virtualization,
which is not available until DRTM launch is performed.  How it works is
explained in some detail in a separate [Qcom-Secure-Launch
repository][qcom-slaunch] as well as in the [theory_of_operation.md
file][too.md] of the main one.  There is no need to repeat all that information
here, but take a look at a picture (taken from Qcom-Secure-Launch) before a few
crucial points are explained:

![Qualcomm DRTM flow](/img/qualcomm-drtm.svg)

These devices boot at EL1 privilege level, which is weird, as normally ARM
systems boot at EL2.  Reaching EL2 makes hardware-assisted virtualization
like KVM possible, which is what slbounce is really meant to address.

slbounce works by running `tcblaunch.exe` at the moment a bootloader (or Linux
with an EFI stub) invokes `ExitBootServices()` before booting into an operating
system (OS).  `tcblaunch.exe` is allowed to run at EL2 because it's signed by
Microsoft and is responsible for initiating DRTM.  If it encounters an error
while processing its input data, `tcblaunch.exe` switches back to EL1 and
invokes an error handler.  Despite going back to EL1, EL2 remains reachable, and
because the handler isn't signed and can execute arbitrary code, it is able to
switch back to EL2.  slbounce does exactly that right before returning from its
version of `ExitBootServices()`, after which an OS boots at EL2.

[slbounce]: https://github.com/TravMurav/slbounce
[qcom-slaunch]: https://github.com/TravMurav/Qcom-Secure-Launch
[too.md]: https://github.com/TravMurav/slbounce/blob/main/theory_of_operation.md

### How to know slbounce has worked

slbounce provides a self-contained binary to verify that it works properly on a
particular hardware called `sltest.efi`.  It essentially reports reaching EL2,
demonstrating that this approach works on this laptop.  Booting a minimal Linux
directly (without GRUB) using `slbounce.efi` also works, with the kernel
reporting that cores run at EL2.  In both cases, that's by itself an indication
of a successful DRTM launch, given that it's a prerequisite for running at EL2
for anything other than `tcblaunch.exe`.

The most direct way of checking for DRTM is looking at the contents of PCRs and
TPM event log.  This part has proven to be unexpectedly difficult.  The device
is not well supported by Linux (or at least not without extra configuration),
so running live distributions or even building a [dedicated Linux
fork][linux-ms] didn't produce anything useful.

However, examining logs on Windows and making Linux dump the start of the TPM
log filled in by a DRTM launch suggests that DRTM PCRs (17-22) get populated.
There is no real reason to suspect a significant difference in behavior with
Linux.

Because the network or drives didn't work in that minimal Linux, extracting
information was hard, and the trick was to write a custom `init` to do the work
of parsing `dmesg` or dumping the TPM event log.  This approach doesn't really
scale, and attempts to get a fully functioning system were abandoned after
realizing the device isn't a good choice for TrenchBoot.  Online information
suggests that Linux can be run on this model, so something must have been
missing.

[linux-ms]: https://github.com/jglathe/linux_ms_dev_kit

## Considerations of using slbounce in TrenchBoot GRUB (or other bootloaders)

The whole point here was evaluating the possibility of leveraging the approach
of slbounce in TrenchBoot.  The bottom line is that it can't be done cleanly.
While some of the limitations can be worked around, the fact of not using an
interface compliant with DEN 0113 would not be possible to ignore in the
affected projects.

Using slbounce requires adopting its requirements and limitations:

- A Windows laptop with a suitable CPU.
- `tcblaunch.exe` that exposes the aforementioned behavior.  Microsoft has
  apparently changed it at some point in 2024 and some versions no longer work,
  although there may be multiple things at play as the range of incompatible
  versions hasn't been established.  The one that did work in tests has
  version `10.0.22621.4317`.  Also, this binary likely can't be redistributed
  as part of TrenchBoot and would have to be copied by users from a Windows
  installation.
- Firmware and `tcblaunch.exe` need to match a few assumptions made by
  slbounce, where it doesn't know details about DRTM launch.  Not everything
  may load successfully in all cases, even though it seems to be relatively
  stable.  The biggest downside is that debugging a case when something doesn't
  work would be rather difficult.

Here are at least some of the assumptions that slbounce makes:

- The size of `tcblaunch.exe` in memory is padded to 2 MiB and assumed to be
  less than that, which apparently (judging by the `git` history) works around
  some issue when the size is smaller.
- Last queried memory map is cached and then used at `ExitBootServices()` under
  the assumption that the memory map hasn't changed.  Maybe the code can be
  updated to query the map; it's not clear why it doesn't.
- `ExitBootServices()` flushes all memory, hoping to avoid a memory corruption
  later on as switching to EL2 discards caches.  This is where the memory map
  is used, and an issue here can result in a crash after DRTM launch is
  performed.

Booting into Linux has been a challenge, and Xen would likely face similar
challenges, if not worse.  Both projects would have to be modified in ways
which wouldn't map onto a real DEN 0113 implementation.

Because slbounce is a driver meant to be used implicitly, there would be no
need to bring its code into TrenchBoot in any form.  Simply loading it at some
point before booting an OS would do.  GRUB, apparently, can't load EFI drivers
now, but it isn't hard to implement, whether loading the driver from a file or
by embedding its data into GRUB.

The code changes necessary for integrating with the slbounce approach would be:

- Moving the code from x86 to some common place and abstracting arch-specific
  behavior.
- Not performing some operations on ARM (or, rather, not adding corresponding
  code for AArch64 and leaving some comments where changes are known to be
  necessary).
- Possibly coming up with a workaround to pass SLRT and implementing it.  This
  could be needed as normally DLMEs get SLRT in a DRTM-specific method, which in
  this case is handled within `tcblaunch.exe`.

### Considerations of emulating DEN 0113 API

Since this device doesn't provide the desired API, one idea was to pretend it's
there by wrapping slbounce to hide its existence from both GRUB and DLMEs.
Unfortunately, not running at EL3 puts severe limitations:

- Running at EL1 before DRTM launch means that SMC calls can't be handled at
  all.
- Because EFI services aren't reliable after DRTM launch, switching to DRTM and
  then doing nothing on an emulated DRTM launch won't work either.  Otherwise,
  it would be possible to handle SMC calls at EL2, but an OS would have to run
  at EL1 to be able to use SMC calls after DRTM launch, which excludes Xen or
  at least leaves it without hardware-assisted virtualization.

Because of the above, the emulation could change the interface to something like
an EFI protocol that works per DEN 0113 but abstracts its SMC calls while
otherwise matching the API:

1. Before a DRTM launch (running at EL1):
    1. The protocol replies with something about capabilities, last error, etc.
2. On a request to perform a DRTM (EL1 -> EL2):
    1. Parse input parameters, validate and preserve them.
    2. Query the memory map; it will be needed after the launch.
    3. Do what slbounce does and reach EL2.
    4. Pass control to the DLME, emulating behavior of DEN 0113 where
       possible (many things won't actually translate to anything, but should
       be able to extend PCRs and pass control with proper contents of
       registers; extending PCRs actually depends on the active locality, which
       isn't known as of now).
    5. If a runtime driver implements the protocol, it may stay in memory,
       provided that DRTM doesn't harm it (maybe it does; this is not known).
3. After a DRTM launch (at EL2):
    1. Handle API calls like "unprotect memory", "close locality", "set error",
       and "enable secure interrupts" (or pretend to handle them).  If the
       protocol doesn't persist, can mock it inside DLME.

Such an implementation wouldn't be a proper API, but switching to it later would
be quite trivial.  Many operations would be a no-op, because required control
over the hardware isn't available.  Still, input and output data would be really
close to DEN 0113 unless external things like the memory map would cause
trouble.

## The search continues

While this Windows-on-Arm device didn't turn out to be a good porting target, it
did help understand what should or shouldn't be present in one:

- a decent compatibility with Linux is a requirement, preferably, out of the
  box: working TPM, storage devices, network
- if DRTM doesn't follow DEN 0113, then access to EL3 is needed to provide a
  compatible implementation
- if a device requires porting firmware, that's also an option; hardware on
  which [ARM Trusted Firmware][tf-a] can already run would be the most
  convenient option
- need an IOMMU providing sufficient DMA protection, but compromises could
  be made in this area in order to facilitate porting

[tf-a]: https://github.com/ARM-software/arm-trusted-firmware/
