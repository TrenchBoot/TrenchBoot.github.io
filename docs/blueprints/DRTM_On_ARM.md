# DRTM on ARM

ARM specifies Dynamic Launch for AArch64 in [ARM DEN 0113, "DRTM Architecture
for Arm"][den0113]. This blueprint records an assessment of that specification,
at version 1.4B, against the TrenchBoot component model. Future revisions will
get dedicated sections or in-place notes depending on the amount of the changes
they bring and their impact.

Authors and reviewers ordered by the time of their first contribution:

- Maciej Pijanowski (3mdeb)
- Sergii Dmitruk (3mdeb)
- Piotr Król (3mdeb)
- Daniel P. Smith (Apertus Solutions, TrenchBoot Project Lead)
- Richard Persaud (EdgeForm)
- Stuart Yoder (Arm)

## Status as of July 2026

At the moment, no TrenchBoot component implements ARM support.

**TrenchBoot is compatible with ARM DRTM, and there are no architectural
obstructions to supporting it.** Every role the project already names has a
direct counterpart in the specification. Each ARM-specific requirement either
has an Intel TXT or AMD SKINIT analog, or amounts to an additional check in
the preamble or the DLME. Compatibility with a UEFI protocol for performing DRTM
is covered under ["Open questions"](#open-questions) below.

## Background

### TrenchBoot

ARM has been named as a future target in project talks since at least 2021:

1. The June 2021 [steering committee meeting][scm-2021] took up "DRTM/TrenchBoot
   for Arm".
2. "ARM Secure Launch Support" appears on the roadmap slide of the LPC 2022
   [TrenchBoot Update][lpc-2022] talk, alongside AMD.
3. The [Xen Project Summit 2024][xen-summit-2024] talk describes TrenchBoot as
   aiming for "a unified approach supporting AMD, Intel and ARM processors".
4. [Secure Launch - DRTM solution on Arm platforms][lpc-2024] was dedicated to
   exploring TrenchBoot on ARM and revealed communication between the two
   parties.
5. The cover letter of the [Secure Launch kernel series][sl-v16], v16, states
   that the series "provides the common infrastructure along with Intel TXT
   support [...] Support for AMD SKINIT is pending the common infrastructure
   getting nailed down, and ARM are looking to build on it too".

[scm-2021]: ../steering-committee/Community-Meeting-June17-2021.md
[lpc-2022]: ../slides/TrenchBoot%20-%20LPC%202022%20-%20Final.pdf
[lpc-2024]: https://lpc.events/event/18/contributions/1814/
[xen-summit-2024]: ../slides/trenchboot_in_xen_2024.pdf

### Beyond TrenchBoot

There is some interest in DRTM on ARM from major companies:

1. [Android Boot, DRTM, UKIs][lpc-2025] by Google and Qualcomm
2. [slbounce][slbounce] works around vendor-locked DRTM on Qualcomm devices
   running Windows, where DRTM is a prerequisite for using hypervisors. This
   means DRTM is utilized on ARM devices despite [documentation][msdn] not
   listing it.

[lpc-2025]: https://lpc.events/event/19/contributions/2257/
[slbounce]: https://github.com/TravMurav/slbounce
[msdn]: https://learn.microsoft.com/en-us/windows/security/hardware-security/how-hardware-based-root-of-trust-helps-protect-windows#requirements-for-qualcomm-processors-with-sd850-or-later-chipsets

### ARM

1. [Linux and DRTM on Arm][lpc-2021-arm] (2021)
2. [DRTM Overview][arm-overview] (2022)
3. [Introduction to Arm DRTM specification and its support in TF-A][arm-video] (2024)

[lpc-2021-arm]: https://lpc.events/event/11/contributions/1120/
[arm-overview]: https://www.trustedfirmware.org/docs/DRTM_Implementation.pdf
[arm-video]: https://www.youtube.com/watch?v=XE0nb0F_Bus

### Secure Launch

The [Secure Launch Specification][spec] v0.5.0 reserves space for ARM without
defining any of it:

- ARM-specific entry tag ID `SLR_ENTRY_ARM_INFO` (`0x0006`, section 4.4.2.2.1)
- `struct slr_entry_arm_info` as a placeholder (section 4.4.5.1)
- "Arm Platforms" marked reserved:
    + in the DLE Handler platform requirements (section 4.1.1)
    + in the SLRT platform requirements (section 4.3)

`SLR_ENTRY_ARM_INFO` may stay empty if there is nothing to put into it. Even
empty entry can be used to identify an ARM boot and to measure SLRT.

[spec]: ../specifications/Secure_Launch.md

The SLRT header's `architecture` field has never had a value enumerated for
ARM; the only two values in use, `SLR_INTEL_TXT` and `SLR_AMD_SKINIT`, are
defined in implementation headers rather than the specification.

### ARM DRTM

ARM has revised its [DEN 0113][den0113] five times so far. `DRTM_VERSION`
reports `major.minor` version only, so it cannot distinguish 1.0 from 1.0B.

| Version | Date       | Changes
| :--:    | :--:       | :--:
| 1.0     | 2023-05-30 | initial release
| 1.0B    | 2024-05-07 | memory region descriptor caching attributes, DLME data diagram
| 1.1     | 2024-10-08 | optional DLME image authentication, DLME Authorities PCR schema
| 1.2     | 2025-07-14 | `DRTM_ENABLE_SECURE_INTERRUPTS` added, TPM buffers zeroed on locality close
| 1.3     | 2025-10-10 | debug and non-secure configuration measurements made mandatory, moved to PCR-18
| 1.4     | 2026-02-25 | relationship to ARM CCA, `FFA_NS_RES_INFO_GET` on FF-A systems
| 1.4B    | 2026-07-21 | relationship to ARM CCA, `FFA_NS_RES_INFO_GET` on FF-A systems

An experimental reference implementation exists in [Trusted Firmware-A][tfa]
(TF-A): the [DRTM service][tfa-svc] in BL31 behind a
[platform porting layer][tfa-plat], described in its [proof of concept][tfa-poc]
design document. It is [disabled by default][tfa-default] (`DRTM_SUPPORT := 0`),
and the only platform port is the Fixed Virtual Platform, which is the sole
consumer of the porting layer ([FVP `platform.mk`][tfa-fvp]; TF-A @`da738d5e`,
2026-05-28).

The reference implementation targets v1.0 with some features left unimplemented,
so it's not conforming, but the core set is there. The implementation gets fixes
and other maintenance but would have to be finished if used on hardware or for
testing in a virtual environment:

- implement system reset on remediation
- implement closing of TPM localities
- if disabling all DMA to start DRTM is not compatible with the hardware, need
  to implement region-based DMA
- `DRTM_ENABLE_SECURE_INTERRUPTS` is missing, but could be omitted until it's
  needed (it's optional for firmware-based DRTM)
- actually extend measurements into TPM instead of simply printing them to the
  console
- if a target device has no dTPM, adding fTPM working as a secure service may be
  necessary
- make all or almost all [DRTM ACS][acs] tests pass, possibly extending the
  tests to cover revisions past v1.2 ([this commit][acs-commit] seems to update
  the suite to v1.2, while [README][acs-readme] still mentions v1.1)
- ACS compiles for EFI using [tianocore/edk2-libc][edk2-libc] and may be able to
  build and work in other environments without requiring significant changes if
  the code is run with sufficiently high privilege level; as an EFI binary, it
  may be executed by GRUB or other EFI bootloaders/applications

[acs]: https://github.com/ARM-software/sysarch-acs/blob/main/docs/drtm/README.md
[acs-commit]: https://github.com/ARM-software/sysarch-acs/commit/919fe830c7021fa78a26cf8e79de1677eee382cf
[acs-readme]: https://github.com/ARM-software/sysarch-acs#drtm-architecture-compliance-suite
[edk2-libc]: https://github.com/tianocore/edk2-libc

## Analysis and comparison

This assessment covers [DEN 0113][den0113] v1.4 in full:

- the component placements it permits
- the SMC interface
- the TPM locality and PCR model
- both DMA protection types
- the DLME entry contract
- the remediation and threat models
- the stated differences from [TCG DRTM][tcg-drtm]

[tcg-drtm]: https://trustedcomputinggroup.org/resource/d-rtm-architecture-specification/

### Compatibility

Each aspect of ARM's specification was checked against what TrenchBoot does
today on Intel TXT and AMD SKINIT. The findings:

- Every component role maps directly, with none left unaccounted for on either
  side and none TrenchBoot would have to invent.
- The DL Event is reached through a defined SMC, which is what the Secure Launch
  Specification's DLE Handler already abstracts, so the invocation model is
  unchanged.
- The requirements new to TrenchBoot are listed under ["Differences that affect
  TrenchBoot"](#differences-that-affect-trenchboot) below. Each is either an
  analog of existing TXT or SKINIT behavior or an additional check; none
  requires a structural change.

The open question is how the launch is triggered, given the UEFI protocol work
on x86. That is a choice of mechanism, not a compatibility obstruction.

### Component mapping

| TrenchBoot                    | ARM DRTM                  | Notes
| :--:                          | :--:                      | :--:
| DCE preamble (GRUB `slaunch`) | DCE preamble              | loads the payload, builds launch parameters, eventually issues the DL Event
| DLE Handler                   | `DRTM_DYNAMIC_LAUNCH` SMC | the invocation interface Secure Launch already abstracts
| D-CRTM (CPU microcode)        | D-CRTM in Secure firmware | resident firmware, or a coprocessor
| DCE (ACM or SKL)              | DCE                       | firmware's responsibility, in BL31 in the reference implementation (TF-A)
| DLME (Linux or Xen)           | DLME                      | entered on the boot PE (Processing Element)
| SLRT                          | DLME data                 | ARM supplies data akin to TXT Heap but leaves space for placing SLRT like AMD

On ARM, the DCE is loaded and authenticated by the D-CRTM, so TrenchBoot does
not supply one as it does on AMD with SKL. Where a Normal world DCE is used
(mostly transparently for TrenchBoot), the preamble allocates a region whose
minimum size `DRTM_FEATURES` advertises.

### What DEN 0113 defines

- AArch64 and TPM 2.0 only, using the TCG crypto-agile event log format.
- The DL Event is an SMC (Secure Monitor Call), `DRTM_DYNAMIC_LAUNCH`
  (`0xC400_0114`), which takes the physical address of a parameter block and
  does not return on success.
- Two implementation classes: firmware-backed, with the D-CRTM in Secure
  firmware, at EL3 alongside the DCE in the reference implementation; and
  hardware-backed, with the D-CRTM in a coprocessor separate from the PE. The
  DCE may also run in the Normal world or be split across both worlds.
- `DRTM_FEATURES` (`0xC400_0111`) makes capabilities discoverable:
    + supported PCR schemas
    + hash algorithm
    + minimum DLME data and Normal world DCE sizes
    + DMA protection types and region counts
    + boot PE identifier
    + TCB (Trusted Computing Base) hash capacity
    + DLME image authentication
- TPM localities are used in sequence and closed as phases complete: the D-CRTM
  initiates at locality 4 and continues at 3, the DCE uses 3, the DLME uses 2.
  Only 2 and 3 can be closed, via `DRTM_CLOSE_LOCALITY` (`0xC400_0115`).
- Under the default PCR schema, PCR-17 holds DCE components and platform state
  while PCR-18 holds the DCE signing key, DLME measurement, and the debug and
  non-secure configuration measurements mandatory since v1.3. PCR-19 to PCR-22
  are reserved for the DLME.
- Two DMA protection types: complete, where the SMMU (System Memory Management
  Unit) blocks all Non-secure device DMA, and region-based, which protects a
  list of regions and permits DMA outside them. A platform supports at least one
  and may advertise both; a launch selects one.
- The scope is the Non-secure side, but the Secure world is not untouched:
  Secure services may have to be quiesced, Secure interrupts may be disabled for
  the launch, and hardware-backed implementations measure TCB components that
  live in the Secure world.

### Relation to TCG DRTM, TXT and SKINIT

[DEN 0113][den0113] borrows TCG DRTM terminology without its implementation
details. DRTM data lives in a single managed memory region, as under SKINIT, and
the launch goes through a defined API rather than a bare instruction with
implicit behavior, as with TXT and PSP-assisted SKINIT. Where ARM departs from
TCG, it has little effect on TrenchBoot:

- PCR usage does not follow the TCG convention, but neither Intel nor AMD do,
  and TrenchBoot does not depend on specific PCR numbers
- localities can be closed, which TXT and PSP-assisted SKINIT also allow
- no TCG data structures other than the event log are used, as in TrenchBoot
- there is no `DLME_Exit` equivalent, so ARM resembles AMD rather than TXT with
  its `SEXIT`
- firmware-based hashing outside the TPM, producing one digest, is the mandatory
  default; TPM-based hashing across every bank is optional and advertised

### DLME hand-off

The DLME is entered on the boot PE with all other PEs off, Non-secure
asynchronous exceptions masked, the requested DMA protections in effect, and TPM
locality 2 active. `X0` holds the physical address of the DLME region and `X1`
the offset from it to the DLME data.

The DLME data is the ARM counterpart of the TXT Heap. It holds:

- the memory map
- the event log
- the list of regions actually protected
- either a TCB hash table or copies of the ACPI tables the DLME consumes, never
  both (the DLME reads which from the `DLME_DATA_HEADER`)

The preamble allocates the DLME data, sized via `DRTM_FEATURES`, and the DCE
populates it, including the event log. The preamble may also use space in the
DLME region outside the image and the data block to pass OS-specific data.
SLRT or a pointer to it can be stored at the start of the DLME region or right
before DLME data, which are the two offsets known from the start. This is
similar to discovering SLRT on AMD.

### Differences that affect TrenchBoot

- In a firmware-backed implementation, the D-CRTM is software, so the dynamic
  root of trust rests on the platform's SRTM in a way it does not on x86.
- Both DMA protection types have to be handled. Under region-based protection
  the DLME region is always protected, but outside it the implementation may
  protect only a subset of the requested regions, so the DLME has to check the
  regions it relies on against the protected region list in the DLME data.
- Errors detected before the D-CRTM modifies system state are returned to the
  caller normally. Once state has changed, the call cannot return, and
  remediation is a persistent error code plus a system reset, read back through
  `DRTM_GET_ERROR` after the reset.
- The DLME validates every ACPI table it consumes against the hash table or
  copies it was given, and must not execute unmeasured code, including UEFI
  Runtime Services, without validating it first.
- Teardown has more steps than x86. Under complete DMA protection, the DLME must
  assume nothing about SMMU state after `DRTM_UNPROTECT_MEMORY` and establish
  its own protections for the OS runtime before handing over, and
  `DRTM_ENABLE_SECURE_INTERRUPTS` has to be called if Secure interrupt
  disabling was requested for the launch.
- The preamble runs at the highest Non-secure exception level: EL2 where a
  hypervisor is present and EL1 otherwise. A hypervisor must trap EL1 SMCs so
  an EL1 caller cannot achieve privilege escalation by starting a DLME at EL2.
- The preamble turns off the secondary PEs and releases active localities, as
  GRUB already does. And where the TPM Command Response Buffer is in normal
  memory, it covers at least the locality 3 buffer in the protection table.

### Work items

A preliminary list:

1. Fill in the ARM placeholders in the Secure Launch Specification:
    + define `struct slr_entry_arm_info`
    + write sections 4.1.1 and 4.3 for ARM
    + enumerate an `architecture` value alongside `SLR_INTEL_TXT` and
      `SLR_AMD_SKINIT` (add them to the specification)
2. Compare the SLRT against the DLME data field by field, deciding for each
   entry whether it is carried in the DLME data, placed alongside it in the DLME
   region, or not needed on ARM.
3. Extend GRUB to act as a DCE preamble on AArch64 (GRUB has most of the code
   present, making it a good target for initial implementation):
    + discover support through `DRTM_VERSION` and `DRTM_FEATURES`
    + halt the secondary PEs if they are running
    + query DRTM capabilities
    + build the parameter block with the memory protection table
    + issue the DL Event
4. Add an AArch64 DLME entry to Linux and Xen alongside the x86 Secure Launch
   entry and handle ARM-specific behavior (e.g., discovery of SLRT, IOMMU
   setup, DMA ranges checks).
5. Set up emulation-based CI, since ARM publishes Fixed Virtual Platforms and
   the Trusted Firmware-A implementation targets one, so unlike the x86
   mechanisms ARM DRTM can be exercised without silicon.
6. Extend DRTM implementation in Trusted Firmware-A if it will be relied upon.
7. After using GRUB for bootstrapping, explore other places for initiating DRTM
   with the aim of reducing the amount of code executed on boot (chunks of the
   GPL2 code could be extracted out of GRUB into a library which then gets used
   in other applications):
    + a minimalistic dedicated EFI application that chainloads Linux or Xen
    + starting DRTM in the firmware (e.g., when coreboot starts EDK2), thus
      making UEFI runtime services safe for use after dynamic launch; see [this
      page][drtm-payload]
    + a new plugin/feature for other bootloaders
8. Extend [meta-trenchboot][meta-trenchboot] to cover AArch64. This
   distribution is meant for demonstration, testing, and development purposes.
   By providing a self-contained and correctly configured environment, it makes
   DRTM-related tasks more deterministic and easier to deal with.
9. [`antievilmaid` component][aem] of Qubes OS is one of the most prominent
   examples of using DRTM in practice. While the implementation targets Qubes
   OS, which is x86-only, none of Qubes OS functionality is required for the
   operation of `antievilmaid`. Making it run outside Qubes OS could be
   utilized on laptops with ARM CPUs.
10. Upstreaming changes to respective projects and participating in maintaining
    them. This is important because the patches are large and non-trivial,
    managing them downstream isn't practical and leads to staying behind
    upstream releases and missing out on important patches and security fixes
    in particular.
11. Extending an existing (e.g., [converged-security-suite][css]) or making a
    new tool for verifying status of DRTM on a platform (supported by firmware
    at all, active or not, checking prerequisites to the extent possible).
    It could be integrated into [HCL reports][tb-hcl] for assessing possibility
    of DRTM on a platform and for evaluating its current state (e.g., checking
    DRTM PCRs, replaying TPM event log to verify their values).
12. Very similar to the point above, but more formal, the implementation should
    target evaluation against requirements found in specifications like
    [HSI][hsi] and [HSTI][hsti].  It remains to be seen whether the set of
    requirements can be vendor-independent and coded for automatic evaluation,
    but this is something worth aiming for.  Either way, the goal is to have
    well-defined criteria and the ability to demonstrate them being fulfilled.
    The more boxes checked, the better security coverage and fewer weaknesses to
    exploit in the design.

Items 1 and 2 need no hardware or virtual environment and can be started
immediately.

[drtm-payload]: https://nlnet.nl/project/Trenchboot-DRTM-launch/
[meta-trenchboot]: https://github.com/zarhus/meta-trenchboot
[aem]: https://github.com/QubesOS/qubes-antievilmaid
[css]: https://github.com/9elements/converged-security-suite
[tb-hcl]: https://github.com/TrenchBoot/trenchboot-hcl/
[hsi]: https://fwupd.github.io/libfwupdplugin/HSI.html
[hsti]: https://learn.microsoft.com/en-us/windows-hardware/test/hlk/testref/hardware-security-testability-specification#hsti-design-considerations

## Open questions

### Secure Launch UEFI protocol

On x86, the UEFI flow hands the launch to Linux: the bootloader registers
the SLRT in the UEFI configuration table and installs the DL stub, which the
kernel's efi-stub calls after `ExitBootServices`. The
[Secure Launch series][sl-v16] added EFI protocol support for the DL stub
callback in v16 (2026-05-15, 38 patches, not merged as of July 2026), so the
DLME can trigger the launch once it is ready.

On ARM, the DLME already receives the protected region list, memory map, event
log and ACPI data through the DLME data, so it may be able to start inside DRTM
instead and not need the callback or the protocol. If the DL stub approach is
used anyway, the constraint to watch is the DLME region layout, since code
using it could overlap a range the preamble needs, for example after unpacking
within the DLME.

This needs resolving before work item 4 and may also affect items 3 and 7.

### Suitable Hardware

Yet to determine which available ARM platforms meet all of the requirements:

- The hardware needs to either support DRTM out of the box or allow the use of
  TF-A as its firmware.
- IOMMU providing sufficient DMA protection is needed (devices that come with
  DRTM should fulfill this requirement).
- Presence of dTPM will remove the need for implementing fTPM in TF-A or
  requiring it from a proprietary firmware, but it's not a hard requirement.
- Hardware-backed implementations provide better security guarantees but must
  be harder to find and, possibly, work with, which plays a role for the
  initial implementation.

What has been looked at:

- [ARM Neoverse RD-V2][neoverse]: DMA protection depends on the IOMMU in use, no
  DRTM
- [Rockchip RK3588][rk3588]: DMA protection may be incomplete, no DRTM
- [NVIDIA DGX Spark][dgx-spark]: not much known about its firmware and
  capabilities, likely to be proprietary, so has to support DRTM out of the box
  to be useful

[neoverse]: https://neoverse-reference-design.docs.arm.com/en/latest/platforms/rdv2.html
[rk3588]: https://rockchips.net/product/rk3588/
[dgx-spark]: https://www.nvidia.com/en-us/products/workstations/dgx-spark/

[den0113]: https://developer.arm.com/documentation/den0113/f
[sl-v16]: https://lore.kernel.org/lkml/20260515211410.31440-1-ross.philipson@gmail.com/
[tfa]: https://github.com/ARM-software/arm-trusted-firmware/
[tfa-default]: https://github.com/ARM-software/arm-trusted-firmware/blob/da738d5eae93af342fdc4995dd3c05acb4c9d757/make_helpers/defaults.mk#L404
[tfa-fvp]: https://github.com/ARM-software/arm-trusted-firmware/blob/da738d5eae93af342fdc4995dd3c05acb4c9d757/plat/arm/board/fvp/platform.mk#L572-L580
[tfa-plat]: https://github.com/ARM-software/arm-trusted-firmware/blob/da738d5eae93af342fdc4995dd3c05acb4c9d757/include/plat/common/plat_drtm.h
[tfa-poc]: https://github.com/ARM-software/arm-trusted-firmware/blob/da738d5eae93af342fdc4995dd3c05acb4c9d757/docs/design_documents/drtm_poc.rst
[tfa-svc]: https://github.com/ARM-software/arm-trusted-firmware/tree/da738d5eae93af342fdc4995dd3c05acb4c9d757/services/std_svc/drtm
