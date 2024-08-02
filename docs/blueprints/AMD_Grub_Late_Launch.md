AMD Grub Late Launcher
======================

## Purpose

The intent of this project is to extend Grub with the ability to call the AMD
SKINIT instruction.

## Background

The AMD SKINIT instruction is a means to initiate a "late launch" that
establishes a Dynamic Root of Trust Measurement (DRTM). The instruction call
requires the system to be in a specific state as enumerated below,

* SVM check, either the `EFER.SVME` bit is set to 1 or the feature flag `CPUID
  Fn8000_0001_ECX[SKINIT]` is set to 1
* The CPU must be in protected mode

## Approach

Grub will be extended with the following capabilities,

* Extend the late launch loader,
    1. determine CPU type and select SKINIT or SENTER path
    2. load kernel (with modules, if applicable) as usual
    3. verify SVM is supported
    4. load Secure Kernel Loader and check if it is valid
    5. allocate and fill Secure Launch Resource Table
    6. send INIT IPI to all APs
    7. disable all TPM localities
* An SKINIT relocator that will,
    1. set protected mode
    2. set registers as required by SKINIT
    3. execute SKINIT as final instruction
