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