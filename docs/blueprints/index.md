# Blueprints

Here you will find the design artifacts for built, in progress, and planned
security engine components:

{nav}

## Components

| Component | DRTM        | Description
| :-------: | :--:        | :---------:
| ACM       | TXT         | DCE for TXT made and signed by Intel
| [GRUB2]   | TXT, SKINIT | Secure Launch enabled bootloader
| [Linux]   | TXT         | Secure Launch enabled Unix-like kernel
| [SKL]     | SKINIT      | Free and open source DCE for AMD SKINIT
| SLRT      | TXT, SKINIT | Data format for sharing information among components
| [Xen]     | TXT, SKINIT | Secure Launch enabled type-1 hypervisor

[GRUB2]: https://github.com/TrenchBoot/grub/
[Linux]: https://github.com/TrenchBoot/linux
[SKL]: https://github.com/TrenchBoot/secure-kernel-loader/
[Xen]: https://github.com/TrenchBoot/xen/

## Boot process

The control passes from a bootloader to DCE and into DLME with each performing
its part:

1. GRUB2
    1. `slaunch` command is used to enable Secure Launch.  It verifies that the
       platform is supported or prints an error and fails.
       In case of TXT, the command also fails if the last Secure Launch attempt
       has failed (the error can result in a soft reset after storing error code
       in a register whose value is analyzed on the next boot).
    2. `slaunch_module` command is run to provide DCEs.  This is always
       needed for SKINIT and sometimes not needed for TXT if BIOS has already
       loaded it.  The command discards unsupported DCEs and thus can be used to
       offer GRUB2 a set of DCEs one at a time and let it figure out which is
       the correct one.
    3. Construct SLRT.
    4. Perform boot using normal GRUB2 commands, although not all of them
       support Secure Launch (see below).  Possibly error if something goes
       wrong during Secure Launch.
2. DCE (ACM or SKL)
    1. Initialize hardware state for DRTM.
    2. Perform TPM measurements.
    3. Start execution of DLME if there are no errors with data or system's
       state in general.
3. DLME (Linux or Xen)
    1. Detect that Secure Launch is in effect.
    2. Find out where SLRT is.
    3. Perform actions specific to a particular DRTM implementation.
    4. Process SLRT (e.g., DRTM measurement policy).
    5. Boot as usual.

## Boot protocols in GRUB2

A summary of currently available implementations:

| DRTM   | Linux | EFI Stub                         | Multiboot2
| :--:   | :---: | :------:                         | :--------:
| TXT    |  Yes  | [On a separate branch][efi-stub] |    Yes
| SKINIT |  Yes  | No                               |    Yes

[efi-stub]: https://github.com/TrenchBoot/grub/tree/grub-sl-2.12-v10

## Upstreaming status (as of 31 August 2024)

Components to which upstreaming doesn't apply were left out from the table.

| Component | Upstream                        | Qubes OS
| :-------: | :------:                        | :------:
| GRUB2     | [Sent][grub-up]                 | [Yes][grub-qos]
| SKL       | Part of Trenchboot              | No
| Linux     | [In Progress][linux-up]         | N/A
| Xen       | [Partially In Progress][xen-ap] | [In Progress][xen-qos]

[grub-up]: https://lists.gnu.org/archive/html/grub-devel/2024-08/msg00088.html
[grub-qos]: https://github.com/QubesOS/qubes-grub2/pull/13
[linux-up]: https://lore.kernel.org/lkml/20240826223835.3928819-1-ross.philipson@oracle.com/
[xen-qos]: https://github.com/QubesOS/qubes-vmm-xen/pull/160
[xen-ap]: https://lists.xenproject.org/archives/html/xen-devel/2023-11/msg01085.html
