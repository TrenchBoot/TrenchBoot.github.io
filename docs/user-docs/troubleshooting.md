# Troubleshooting

Covering all the possible reasons why a Secure Launch might fail is beyond the
scope of this document.  It only suggests what can be checked in an attempt to
diagnose the issue and simple ways to address some situations.

## Firmware settings

If problems are encountered, the first thing to check is the firmware setting
on the system:

- Is the TPM enabled?
- On Intel: are VTx and VTd enabled?
- On AMD: is SVM enabled?
- Is DRTM (TXT/SKINIT) enabled?

## Failed start with Intel TXT

TXT provides a sticky error register that will contain the last error coming
from TXT or the Secure Launch kernel/hypervisor.  If there was an error, the
contents of TXT error register is printed by `slaunch` GRUB command from within
GRUB shell after a soft reboot.  Drop into the shell by typing `c` at the GRUB
menu and run:

```text
grub> slaunch
TXT_ERRORCODE reports failure: 0xc0008001
```

`slaunch_state` dumps more detailed information about the state of TXT.
However, the command is of any use only after a successful `slaunch`:

```text
grub> slaunch
grub> slaunch_state
Secure launcher: Intel TXT
  TXT.STS: 0x0000000000004092
    SENTER.DONE.STS:        0
    SEXIT.DONE.STS:         1
    MEM-CONFIGLOCK.STS:     0
    PRIVATEOPEN.STS:        1
    TXT.LOCALITY1.OPEN.STS: 0
    TXT.LOCALITY2.OPEN.STS: 0
  TXT.ESTS: 0x00
    TXT_RESET.STS: 0
  TXT.E2STS: 0x0000000000000000
    SECRETS.STS: 0
  TXT.ERRORCODE: 0x00000000
  TXT.DIDVID: 0x00000001b0078086
    VID:    0x8086
    DID:    0xb007
    RID:    0x0001
    ID-EXT: 0x0000
  TXT.VER.FSBIF: 0xffffffff
  TXT.VER.QPIIF: 0x9d003000
    DEBUG.FUSE: 1
  TXT.SINIT.BASE: 0x77ec0000
```

In the second case the error is `0x0000000` meaning there was no previous error
because `SENTER` command wasn't used recently.  An error of the form
`0xc0008XXX` is coming from the Secure Launch kernel code.  The error codes are
detailed in the Linux documentation and listed in the
[main header file][slaunch.h].

Errors coming from other sources like the CPU or the SINIT ACM have different
forms.  Consult the TXT documentation [from Intel][intel-txt] to determine what
the error means.

Because the error code can prevent the use of Secure Launch and is only
preserved by soft reboots, a hard reboot or a power off followed by a power on
can sometimes be necessary.

[slaunch.h]: https://github.com/TrenchBoot/linux/blob/linux-sl-master-9-12-24-v11/include/linux/slaunch.h#L108
[intel-txt]: https://software.intel.com/en-us/articles/intel-trusted-execution-technology

## TPM operations suddenly start to fail

This may be caused by the TPM safety timer.  If your machine was not safely shut
down (e.g., due to a power loss) and it had been running for less than
approximately 70 minutes (2<sup>22</sup> ms) since, you may experience such an
issue.  Simply try to reboot the machine at a later time.

## Getting errors from GRUB

By default, GRUB doesn't print any internal errors.  This can be changed by
setting `debug` variable to a list of interesting components (as a comma- or
whitespace-separated list):

```text
set debug=slaunch
```

See GRUB's documentation for some more details.
