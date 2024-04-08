# Hardware test matrix

Devices and configurations TrenchBoot has been tested on.

| Tested devices                                                               | TPM family | Result | Notes |
|:----------------------------------------------------------------------------:|:----------:|:------:|:-----:|
| Intel Tiger Lake client                                                      |  TPM 2.0   |  PASS  | UEFI firmware |
| Intel Kaby Lake server                                                       |  TPM 2.0   |  PASS  | UEFI firmware |
| Intel Skylake sever                                                          |  TPM 2.0   |  PASS  | UEFI firmware |
| Dell OptiPlex 9010                                                           |  TPM 1.2   |  PASS  | coreboot SeaBIOS firmware |
| PC Engines APU2 platform series <br>(AMD family 16h models 30h-3fh embedded) |  TPM 2.0   |  PASS  | coreboot firmware |
| PC Engines APU2 platform series <br>(AMD family 16h models 30h-3fh embedded) |  TPM 1.2   |  PASS  | coreboot firmware |
| Asus KGPE-D16 <br>(AMD Opteron family 15h models 00h-0fh server)             |  TPM 1.2   |  PASS  | stock BIOS |
| Asus KGPE-D16 <br>(AMD Opteron family 15h models 00h-0fh server)             |  TPM 2.0   |  FAIL  | coreboot firmware TPM issue |
| Supermicro M11SDV-8CT <br>(AMD EPYC 3000 Snowy Owl server)                   |  TPM 2.0   |  PASS  | CSM legacy boot |
| Supermicro M11SDV-8CT <br>(AMD EPYC 3000 Snowy Owl server)                   |  TPM 2.0   |  FAIL  | UEFI boot |

Hardware quirks and workarounds.

| Device                                                                       | Notes |
|:----------------------------------------------------------------------------:|:-----:|
| Dell OptiPlex 9010                                                           | 1. Installer has issues rebooting without `reboot=pci` kernel option. |
| (continued)                                                                  | 2. Xen sometimes has issues rebooting, boot cycle is the workaround. |
| Asus KGPE-D16 <br>(AMD Opteron family 15h models 00h-0fh server)             | IOMMU has no extended features: can't use `INVALIDATE_IOMMU_ALL` in SKL. |
| Supermicro M11SDV-8CT <br>(AMD EPYC 3000 Snowy Owl server)                   | Problematic USB controller for Qubes OS ([resets the system][qubesos-m11-reset]). |
| (continued)                                                                  | Works without `sys-usb` VM or if USB controller is disabled. |

[qubesos-m11-reset]: https://github.com/QubesOS/qubes-issues/issues/8322#issuecomment-1904423204
