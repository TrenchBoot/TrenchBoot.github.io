# TrenchBoot Use Cases

TrenchBoot is meant to be a universal framework to enable building integrity in
the launch process of systems. To relate to real world usage, it is good to
have a set of use cases that explain a subset of situations where TrenchBoot is
applicable and how it would work in those situations. Below are a series of use
cases that are actively being investigated and/or worked on.

## Crowd Sourcing Integrity

There is currently no known public authority available to verify BIOS/Firmware
PCR values. TrenchBoot would like to become such an authority but there is the
challenge of how to obtain all these values in a manner that provide assurance
to the authenticity of the values. Crowd sourcing provides the best means to
collect the largest and most diverse set of values. The challenge with crowd
sourcing the values is how to establish authenticity of the values. This
challenge can be overcome with a TrenchBoot based live CD that establishes an
attestation identity provisioned by a TrenchBoot Attestation Certificate
Authority (ACA).

## Network Attestation Launch

An individual or enterprise may not want to allow a system to boot on their
network unless it is running a known configuration. When TrenchBoot is
installed onto a system it will work in conjunction with a TrenchBoot ACA
(public or private instance) that provides a key management service. TrenchBoot
will hold a portion of a Shamir Secret Sharing key with another portion held by
the key management service. For the system to boot it will attest to the key
management service to obtain a key fragment that will allow it to unlock the system
disk.

## Travel Laptop 2FA Launch

When travelling there are times when an individual loses positive control of
their device. During these times attackers can launch physical access attacks.
For this configuration TrenchBoot will "double chain wrap" the encryption key
for decrypting the system where each chain wrap correlates to an authentication
factor. Working internal to external, the system drive key is encrypted with
the first wrap key that is in turned encrypted with the second wrap key. The
first wrap key is stored on a removable token device, e.g. YubiKey, and the
second wrap key is sealed in a TPM NVRAM slot. For a system to boot it must
have launched with the correct firmware and the token must be present.

## Cross-links to publications mentioning TrenchBoot

### Blog posts

- [Open Source DRTM with TrenchBoot for AMD processors. Introduction.](
    https://blog.3mdeb.com/2020/2020-03-28-trenchboot-nlnet-introduction/)
- [TrenchBoot - Open Source DRTM for AMD processors. Project's basics.](
    https://blog.3mdeb.com/2020/2020-03-31-trenchboot-nlnet-lz/)
- [TrenchBoot: Open Source DRTM. Landing Zone validation.](
    https://blog.3mdeb.com/2020/2020-04-03-trenchboot-nlnet-lz-validation/)
- [TrenchBoot: Open Source DRTM. DRTM update and meta-trenchboot implementation](
    https://blog.3mdeb.com/2020/2020-04-30-trenchboot-nlnet-release-04/)
- [TrenchBoot: Open Source DRTM. CI/CD system.](
    https://blog.3mdeb.com/2020/2020-05-05-trenchboot-nlnet-ci-cd-system/)
- [Installing TrenchBoot in UEFI environments](
    https://blog.3mdeb.com/2020/2020-05-06-trenchboot-uefi-environment/)
- [Starting TrenchBoot's Landing Zone from iPXE](
    https://blog.3mdeb.com/2020/2020-06-01-ipxe_lz_support/)
- [TrenchBoot: Open Source DRTM. GRUB's new features and TPM event log.](
    https://blog.3mdeb.com/2020/2020-07-03-trenchboot-grub-cbfs/)
- [DEV and IOMMU: a story of two DMA protection mechanisms](
    https://blog.3mdeb.com/2020/2020-07-03-dev_and_iommu/)
- [TrenchBoot: Open Source DRTM. TPM event log all the way.](
    https://blog.3mdeb.com/2020/2020-08-13-trenchboot-event-log/)
- [TrenchBoot: Open Source DRTM. Multiboot2 support.](
    https://blog.3mdeb.com/2020/2020-09-07-trenchboot-multiboot2-support/)
- [TrenchBoot: Xen hypervisor support for the TrenchBoot](
    https://blog.3mdeb.com/2020/2020-10-15-xen-implementation-for-trenchboot/)
- [Proof of concept implementation of RATS attestation for the TrenchBoot](
    https://blog.3mdeb.com/2020/2020-12-14-trenchboot_attestation/)
- [TrenchBoot Anti Evil Maid for Qubes OS](
    https://blog.3mdeb.com/2023/2023-01-31-trenchboot-aem-for-qubesos/)
- [TrenchBoot Anti Evil Maid - Phase 2](
    https://blog.3mdeb.com/2023/2023-09-27-aem_phase2/)
- [TrenchBoot Anti Evil Maid - Phase 3](
    https://blog.3mdeb.com/2024/2024-01-12-aem_phase3/)
- [TrenchBoot Anti Evil Maid - Phase 4](
    https://blog.3mdeb.com/2024/2024-04-11-aem_phase4/)

### Scientific Publications

- [Mathias Schüpany, Martin Pirker: A Revisit of Attestable Nodes for
    Networked Applications](https://dl.acm.org/doi/abs/10.1145/3538969.3544433)
- [Jasmin Marmsoler, Thomas Grechenig, Florian Fankhauser: TPM 2.0 als
    Sicherheitsmaßnahme gegen Rootkits auf Linux-basierten Desktop-Systemen](
    https://scholar.archive.org/work/kszetgytmjhglic5cngst5dnle) (German)

### Books

- [Jiewen Yao, Vincent Zimmer: Building Secure Firmware](
    https://link.springer.com/book/10.1007/978-1-4842-6106-4)
