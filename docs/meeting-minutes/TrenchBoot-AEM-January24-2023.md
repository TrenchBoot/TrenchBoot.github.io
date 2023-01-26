## Attendees

- Daniel P. Smith (Apertus Solutions)
- Andrew Cooper (Citrix)
- Krystian Hebel (3mdeb)
- Michał Żygowski (3mdeb)

## Agenda

- Review the current state of TrenchBoot boot protocol being upstreamed to GRUB
  and Linux kernel
- Discuss the scope of the work needed to align Qubes OS Anti Evil Maid
  solution based on TrenchBoot with the most recent boot protocol version
- Discuss possibilities to implement DRTM launch on S3 resume in Xen

## Meeting Minutes

Current approach of booting Linux with TrenchBoot:

- meet Linux demand so that anything can do EBS and DRTM can still work
- Linux doing EBS to gather necessary info from firmware
- bootloader to put slaunch code to memory
- the code being put to memory is going to be standardized and does everything
  what DRTM technology needs
- standardize an input table to be used for generic stub not specific to any
  bootloader/kernel

Current Xen status:

- Xen efi needs fixing in order to work without Xen's efi-stub, so it would be
  better to follow Linux's approach of jumping into Xen and Xen calling out to
  slstub
- AP wakeup developed during the Qubes OS AEM PoC in Xen hypervisor kernel is
  still valid and will be needed in the most recent approach as well
- for legacy boot module will have to be placed in memory by GRUB but GRUB will
  also construct the input table and call SENTER/SKINIT

DRTM S3 resume support for Xen:

- Idea: for legacy S3 resume mark pages with slaunch module so that Xen won't
  use them and later do DRTM relaunch using the module (needs a plan how to
  implement it in Xen for legacy BIOS boot)
- tboot relaunches itself and then resumes kernel, seals its state to DRTM
  PCRs, does DRTM on itself and tries to unseal the state, if it is ok, the
  state is unsealed and tboot may proceed with kernel resume (not exactly what
  we want)
- Xen patches alternatives, not runtime, S3 resume measurement would not be the
  same even when only text section would be measured
- ideal solution is for Xen to reinstantiate itself with DRTM and reload the VM
  configuration

Possible solution for S3:

- boot into Xen
- search for SRT table and look for SL module
- take measurement of Xen at the end of boot and by delaying the dynamic launch
  until after we're done making boot-time modifications to Xen's .text/etc. so
  that the measurement will be stable. This doesn't cope with livepatching yet,
  but livepatching support can be omitted in v1 implementation.
- Currently it is not possible to take a meaningful measurement of Dom0 after
  coming back from S3. It is still valid to measure Dom0 on initial boot to
  ensure that the correct dom0 kernel is used to start the system. Also dom0
  measurement will be largely meaningless, so it can be excluded from v1
  implementation
