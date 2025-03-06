# Hardware test matrix

## Modern (as of 2024) test results

These results are based on the use of TrenchBoot in combination with [AEM][aem]
on the specified hardware running [Qubes OS][qubesos].  The tests were carried
out either automatically via [openQA][openqa] using [this test][aem-test] or
manually by [installing][aem-install] and [verifying][aem-verify] AEM with
TrenchBoot.  The procedure is very similar in both cases:

1. Installation of ACMs on Intel platforms or SKL on AMD platforms.
2. Installation of TrenchBoot's versions of AEM, Xen and GRUB2 (necessary until
   all of the changes are upstreamed).
3. AEM setup procedure which is detailed in its [README][aem-readme].
4. Reboot so that AEM has a chance to seal the secrets taking values of DRTM
   PCRs into account.
5. Reboot once again to verify that AEM has successfully unsealed the secrets
   and PCR values are correct with respect to TPM event log.
6. An extra manual step in both cases (openQA only prints event log) is to
   check that hashes from the log correspond to on-disk files.

Devices and configurations on which TrenchBoot is known to work (availability
years can be approximate; automatic tests don't override manual tests):

| Tested device                                                                                 | TPM family | Available  | Notes |
|:---------------------------------------------------------------------------------------------:|:----------:|:----------:|:-----:|
| Asus KGPE-D16<br>(AMD Opteron family 15h models 00h-0fh server)<br>(v0.4 by [@SergiiDmytruk]) |  TPM 1.2   | 2005-2015  | stock BIOS |
| Dell OptiPlex 9010<br>(v0.3 by openQA)                                                        |  TPM 1.2   | 2012-2017  | coreboot SeaBIOS firmware, bad 1st entry in event log |
| HP Thin Client t630<br>(v0.4 by [@krystian-hebel])                                            |  TPM 2.0   | 2016-2020  | CSM legacy boot<br>BIOS updates in 2024 |
| HP Thin Client t630<br>(v0.5 by openQA)                                                       |  TPM 1.2   | 2016-2020 | legacy boot and UEFI boot |
| HP Thin Client t630<br>(v0.5 by openQA)                                                       |  TPM 2.0   | 2016-2020 | legacy boot and UEFI boot |
| Dell OptiPlex 7010<br>(v0.5 by openQA)                                                        |  TPM 1.2   | 2012-2017 | legacy boot and UEFI boot |
| Protectli VP4670<br>(v0.5 by openQA)                                                          |  TPM 2.0   | 2022-today | legacy boot and UEFI boot |

Devices and configurations on which TrenchBoot is known to not work:

| Tested device                                                                                 | TPM family | Available  | Notes |
|:---------------------------------------------------------------------------------------------:|:----------:|:----------:|:-----:|
| Supermicro M11SDV-8CT<br>(AMD EPYC 3000 Snowy Owl server)<br>(v0.4 by openQA)                 |  TPM 2.0   | 2019-today | CSM legacy boot,<br>unexpected PCR values |

[aem]: https://github.com/TrenchBoot/qubes-antievilmaid
[qubesos]: https://www.qubes-os.org/
[openqa]: https://open.qa/
[aem-test]: https://github.com/TrenchBoot/openqa-tests-qubesos/blob/3mdeb-lab/tests/aem_hw.pm
[aem-install]: https://blog.3mdeb.com/2023/2023-09-27-aem_phase2/#installation-procedure
[aem-verify]: https://blog.3mdeb.com/2024/2024-01-12-aem_phase3/#testing
[aem-readme]: https://github.com/QubesOS/qubes-antievilmaid/blob/main/README
[@SergiiDmytruk]: https://github.com/SergiiDmytruk
[@krystian-hebel]: https://github.com/krystian-hebel

## Legacy (before 2022) test results

The origin of the following results is not always known.  AMD platforms used to
be tested on [CI][ci] via [testing-trenchboot].

Devices and configurations on which TrenchBoot is known to work (availability
years can be approximate):

| Tested device                                                               | TPM family | Available  | Notes |
|:---------------------------------------------------------------------------:|:----------:|:----------:|:-----:|
| Intel Kaby Lake server                                                      |  TPM 2.0   | 2016-2020  | UEFI firmware |
| Intel Skylake server                                                        |  TPM 2.0   | 2015-2019  | UEFI firmware |
| Intel Tiger Lake client                                                     |  TPM 2.0   | 2020-2023  | UEFI firmware |
| PC Engines APU2 platform series<br>(AMD family 16h models 30h-3fh embedded) |  TPM 1.2   | 2016-2023  | coreboot firmware |
| PC Engines APU2 platform series<br>(AMD family 16h models 30h-3fh embedded) |  TPM 2.0   | 2016-2023  | coreboot firmware |

Devices and configurations on which TrenchBoot is known to not work:

| Tested device                                                               | TPM family | Notes |
|:---------------------------------------------------------------------------:|:----------:|:-----:|
| [Asus KGPE-D16][kgpe]<br>(AMD Opteron family 15h models 00h-0fh server)     |  TPM 2.0   | coreboot firmware TPM issue |
| [Supermicro M11SDV-8CT][m11]<br>(AMD EPYC 3000 Snowy Owl server)            |  TPM 2.0   | UEFI boot |

[ci]: https://gitlab.com/trenchboot1/3mdeb/meta-trenchboot/-/pipelines
[testing-trenchboot]: https://github.com/3mdeb/testing-trenchboot
[kgpe]: https://github.com/TrenchBoot/trenchboot-issues/issues/27
[m11]: https://github.com/TrenchBoot/trenchboot-issues/issues/28

## Hardware quirks and workarounds

These are difficulties/things of note one has to face when using these platforms
today. They were probably more usable years ago, but something has changed in
software that nobody tested on these devices and now experience isn't very
smooth.

| Device                                                                      | Notes |
|:---------------------------------------------------------------------------:|:-----:|
| Asus KGPE-D16<br>(AMD Opteron family 15h models 00h-0fh server)             | IOMMU has no extended features: can't use `INVALIDATE_IOMMU_ALL` in SKL. |
| Dell OptiPlex 9010                                                          | 1. Installer has issues rebooting without `reboot=pci` kernel option. |
| (continued)                                                                 | 2. Xen sometimes has issues rebooting, boot cycle is the workaround. |
| (continued)                                                                 | 3. First entry in DRTM TPM event log contains the result of PCR extend operation instead of its input due to ACM bug. |
| HP Thin Client t630                                                         | Starting Qubes OS installation in legacy mode requires extra steps ([see][qubesos-t630-install]). (Not required since Qubes R4.2.3)|
| Supermicro M11SDV-8CT<br>(AMD EPYC 3000 Snowy Owl server)                   | Problematic USB controller for Qubes OS ([resets the system][qubesos-m11-reset]). |
| (continued)                                                                 | Works without `sys-usb` VM or if USB controller is disabled. |

[qubesos-m11-reset]: https://github.com/QubesOS/qubes-issues/issues/8322#issuecomment-1904423204
[qubesos-t630-install]: https://github.com/TrenchBoot/TrenchBoot.github.io/pull/30#discussion_r1570519887
