# TXT Grub Late Launcher

## Purpose

The intent of this project is to extend Grub with the ability to call
the Intel SENTER instruction.

## Background

The Intel SENTER instruction is a means to initiate a "late launch"
that establishes a Dynamic Root of Trust Measurement (DRTM). The
instruction call requires the system to be in a specific state as
enumerated below,

## Approach

Grub will be extended with the following capabilities,

- A late launch loader that will,
  1. verify SENTER is supported
  1. load ACM and verify it matches platform
  1. build pagetable for MLE
  1. set types of cache as required by TXT
  1. enable native FPU error reporting
  1. verify no machine check in progress
  1. parse GETSEC\[PARAMETERS\]
  1. clear machine check registers
  1. allocate and fill Secure Launch Resource Table
- An SENTER relocator that will,
  1. set protected mode
  1. set registers as required by SENTER
  1. execute GETSEC\[SENTER\] as final instruction
