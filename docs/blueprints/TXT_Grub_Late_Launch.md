TXT Grub Late Launcher
======================

## Purpose

The intent of this project is to extend Grub with the ability to call the Intel
SENTER instruction.

## Background

The Intel SENTER instruction is a means to initiate a "late launch" that
establishes a Dynamic Root of Trust Measurement (DRTM). The instruction call
requires the system to be in a specific state as enumerated below,

## Approach

Grub will be extended with the following capabilities,

* A late launch loader that will,
    1. verify SENTER is supported
    2. load ACM and verify it matches platform
    3. build pagetable for MLE
    4. set types of cache as required by TXT
    5. enable native FPU error reporting
    6. verify no machine check in progress
    7. parse GETSEC[PARAMETERS]
    8. clear machine check registers
    9. allocate and fill Secure Launch Resource Table
* An SENTER relocator that will,
    1. set protected mode
    2. set registers as required by SENTER
    3. execute GETSEC[SENTER] as final instruction
