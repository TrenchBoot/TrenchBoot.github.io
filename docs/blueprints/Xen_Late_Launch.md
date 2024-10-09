# Xen Late Launch Hypervisor

## Summary

Xen hypervisor was extended to make it function as the late launch hypervisor
for both AMD SKINIT and Intel TXT.  The changes have not yet been upstreamed and
currently target packages for Qubes OS.

## Background

The late launch process provides means to measure a target execution
environment as prescribed by the CPU's late launch protocol before jumping into
the environment. Thereby establishing a hardware-based Root of Trust for
Measurement (RTM).

## Approach

File format changes:

* Added an MLE header (defined for Intel TXT but usable on AMD where DCE is
    controlled by us) to provide basic information to the bootloader including
    an entry point
* Added a new entry point specific for the late launch to know how Xen was
    started and be able to change behaviour based on that knowledge

Control flow changes:

1. Start Xen from GRUB2 via Multiboot2 protocol but through the entry point
    from MLE header
2. Figure out location of SLRT and MBI:
    - [TXT] Extract addresses from OS2MLE structure
    - [SKINIT] Addresses are available in `%EBP` and `%EBX` registers
    respectively
3. [TXT] Verify that PMR ranges are sane and provide protections for all
    involved components
4. Reserve memory to protect it from being overwritten:
    - memory range used by TPM MMIO
    - [TXT] memory occupied by TXT heap and ACM
    - [SKINIT] memory occupied by SLB
5. [TXT] Restore pre-RTM values of MTRRs
6. Measure MBI
7. Process DRTM policy from SLRT (e.g., parts of SLRT, Linux kernel, its
    parameters, memory map, initrd)

Other code modifications:

* Add TXT-specific way of waking up APs
* Perform parallel startup of APs, because TXT always starts them
    simultaneously
* Add TPM driver and update TPM event log when PCRs are extended
