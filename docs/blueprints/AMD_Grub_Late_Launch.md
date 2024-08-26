# AMD Grub Late Launcher

## Purpose

The intent of this project is to extend Grub with the ability to call
the AMD SKINIT instruction.

## Background

The AMD SKINIT instruction is a means to initiate a "late launch" that
establishes a Dynamic Root of Trust Measurement (DRTM). The
instruction call requires the system to be in a specific state as
enumerated below,

- SVM check, either the `EFER.SVME` bit is set to 1 or the feature
  flag `CPUID Fn8000_0001_ECX[SKINIT]` is set to 1
- The CPU must be in protected mode

## Approach

Grub will be extended with the following capabilities,

- Extend the late launch loader,
  1. determine CPU type and select SKINIT or SENTER path
  1. load kernel (with modules, if applicable) as usual
  1. verify SVM is supported
  1. load Secure Kernel Loader and check if it is valid
  1. allocate and fill Secure Launch Resource Table
  1. send INIT IPI to all APs
  1. disable all TPM localities
- An SKINIT relocator that will,
  1. set protected mode
  1. set registers as required by SKINIT
  1. execute SKINIT as final instruction
