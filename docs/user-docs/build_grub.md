# Building and installing GRUB

At the time of writing (August 2024) EFI and legacy changes are on different
branches in the same repository and neither builds on top of the other in
terms of commit history.  Either clone the repository in full and switch
between branches to build both versions or use shallow clones as shown below.

If your system isn't setup for development, you might want to use
[trenchboot-sdk] Docker container.  It can be run like this to perform a build:

```shell
docker run --rm -it -v "$PWD:$PWD" -w "$PWD" \
           -e HOME="$PWD/home" \
           --user "$(id -u):$(id -g)" \
           ghcr.io/trenchboot/trenchboot-sdk:master /bin/bash
```

Setting `$HOME` is necessary because GRUB will try to use `ccache` which fails
if there is no `$HOME`.

[trenchboot-sdk]: https://github.com/TrenchBoot/trenchboot-sdk

## EFI version

### Build

```bash
# check out the sources
git clone --branch grub-sl-2.12-v9 --depth 1 \
          https://github.com/TrenchBoot/grub.git
cd grub

# make the checkout buildable
./bootstrap

# configure
mkdir build
cd build
../configure --with-platform=efi --target=x86_64

# build
make -j$(nproc)
```

### Setup

```bash
# also from inside build/ directory
echo 'configfile ${cmdpath}/grub.cfg' > embedded.cfg
./grub-mkimage -O x86_64-efi -o grubx64.efi -p / -c embedded.cfg -d grub-core \
               all_video boot btrfs cat chain configfile echo efifwsetup \
               efinet ext2 fat font gfxmenu gfxterm gzio halt hfsplus iso9660 \
               jpeg loadenv loopback lvm mdraid09 mdraid1x minicmd normal \
               part_apple part_msdos part_gpt password_pbkdf2 png reboot \
               regexp search search_fs_uuid search_fs_file search_label serial \
               sleep syslinuxcfg test tftp video xfs backtrace http linux usb \
               usbserial_common usbserial_pl2303 usbserial_ftdi \
               usbserial_usbdebug keylayouts at_keyboard multiboot2
```

The first command creates an embedded config which sources grub.cfg in the
directory in which GRUB image is located.

The second command produces the UEFI GRUB image named `grubx64.efi`.  The image
along with a suitable `grub.cfg` needs to be installed on a EFI System
Partition (ESP) and pointed at so firmware can find it (via Setup UI or a tool
like `efibootmgr`).  It can also be started manually from a UEFI shell for a
test.

## Legacy version

### Build

```bash
# check out the sources
git clone --branch tb-2.12-57-v1 --depth 1 https://github.com/TrenchBoot/grub.git
cd grub

# make the checkout buildable
./bootstrap

# configure
mkdir build
cd build
../configure --prefix=$PWD/local-install --target=i386

# build
make -j$(nproc)
```

### Setup

Installation for legacy boot from inside of a container requires access to
device files and mounting of boot directory.  For this reason the container
needs to be started as `root` (inside of the container, don't prepend `sudo` to
the command) and be a privileged one:

```bash
docker run --rm -it -v "$PWD:$PWD" -w "$PWD" \
           --user root:root --privileged \
           ghcr.io/trenchboot/trenchboot-sdk:master /bin/bash
```

```bash
# if done in a container
cd build
mount BOOT-PARTITION PATH-TO-BOOT

# also from inside build/ directory
make install
local-install/sbin/grub-install --target=i386-pc \
                                --boot-directory PATH-TO-BOOT \
                                DISK-DEVICE
```

In the second command above replace:

- `BOOT-PARTITION` with the `/dev/...` file corresponding to the boot partition
- `PATH-TO-BOOT` with the path to where the boot partition is mounted (`grub/`
  directory will be created there)
- `DISK-DEVICE` with the `/dev/...` file corresponding to the drive (not a
  partition) on which GRUB is to be installed (double check before running the
  command to avoid accidentally overwriting some working GRUB setup)

If you get a weird error like ``error: disk `hostdisk//dev/sda1' not found.``,
it means that GRUB doesn't have write permissions for device files.

## Configuration

There is a new GRUB command, which instructs GRUB to initiate a Secure Launch
when booting an OS, called `slaunch`.  This is an example of a GRUB menuentry
that would be used to do a Secure Launch of the Linux kernel:

```text
menuentry 'Linux with Secure Launch 6.8.0-rc3-master-v8' --unrestricted {
        load_video
        insmod gzio
        insmod part_gpt
        insmod xfs
        search --no-floppy --fs-uuid --set=root bba24662-776e-4396-9b1e-9ee5606d79b8
        slaunch
        slaunch_module /dce-for-a-given-platform
        linux /vmlinuz-6.8.0-rc3-master-v8 root=/dev/mapper/root ro crashkernel=auto resume=/dev/mapper/swap rd.lvm.lv=my/root rd.lvm.lv=my/swap rhgb console=ttyS0,115200n8 console=tty0 LANG=en_US.UTF-8
        initrd /initrd-6.8.0-rc3-master-v8.img
}
```

Note also the optional `slaunch_module` command which tells GRUB to load an
external SINIT ACM for this configuration.  In general, server
platforms contain an existing SINIT ACM in the firmware and this line can be
omitted.  For client platforms, an external ACM is required to be supplied.  The
SINIT ACM for a given platform can be acquired [from Intel][intel-acm].

[intel-acm]: https://www.intel.com/content/www/us/en/developer/articles/tool/intel-trusted-execution-technology.html

## Validation

There are a number of ways to validate that a successful Secure Launch was
done.  Using serial logging or `dmesg`, search for the string "TXT" after
booting:

```shell
[root@my-system ~]# dmesg | grep TXT
[    0.000094] slaunch: Intel TXT setup complete
[    2.617782] slaunch: TXT AP startup vector address updated
```

That indicates a successful Secure Launch boot.  Another way is to display the
Secure Launch TPM event log.  This can be done as follows after booting (note
only the tail end of the log is shown here for brevity, the rest is snippped):

```shell
[root@my-system ~]# cat /sys/kernel/security/slaunch/eventlog | hexdump -C
...
[snip]
...
00000490  a3 e2 de 6b fb 1f 79 ef  c9 5e de bf ef bf 92 fb  |...k..y..^......|
000004a0  fc b2 89 ea 64 c1 d7 d2  99 fb 49 e6 12 00 00 00  |....d.....I.....|
000004b0  4d 65 61 73 75 72 65 64  20 53 4c 52 20 54 61 62  |Measured SLR Tab|
000004c0  6c 65 12 00 00 00 02 05  00 00 01 00 00 00 0b 00  |le..............|
000004d0  cd 64 bf e1 70 96 4c ce  53 2f 2f 7a 85 85 fe f0  |.d..p.L.S//z....|
000004e0  05 22 40 f6 62 18 bf 94  2a 2f 3d 14 b1 25 60 31  |."@.b...*/=..%`1|
000004f0  18 00 00 00 4d 65 61 73  75 72 65 64 20 62 6f 6f  |....Measured boo|
00000500  74 20 70 61 72 61 6d 65  74 65 72 73 11 00 00 00  |t parameters....|
00000510  02 05 00 00 01 00 00 00  0b 00 18 7d 80 8f 2c ca  |...........}..,.|
00000520  03 bf a7 54 ff 1d 16 6d  49 51 25 f6 bc ec 46 dc  |...T...mIQ%...F.|
00000530  23 a7 39 a8 db 96 28 8e  d4 1d 16 00 00 00 4d 65  |#.9...(.......Me|
00000540  61 73 75 72 65 64 20 4b  65 72 6e 65 6c 20 69 6e  |asured Kernel in|
00000550  69 74 72 64 12 00 00 00  02 05 00 00 01 00 00 00  |itrd............|
00000560  0b 00 11 02 09 6f c6 1d  78 11 87 1a 93 49 10 2f  |.....o..x....I./|
00000570  14 69 dd 45 b8 c3 03 e7  e6 80 6e 21 9b 87 47 90  |.i.E......n!..G.|
00000580  d6 27 1c 00 00 00 4d 65  61 73 75 72 65 64 20 4b  |.'....Measured K|
00000590  65 72 6e 65 6c 20 63 6f  6d 6d 61 6e 64 20 6c 69  |ernel command li|
000005a0  6e 65 12 00 00 00 02 05  00 00 01 00 00 00 0b 00  |ne..............|
000005b0  b2 29 3f 3c da 25 4a 78  61 be 76 91 3e 06 f9 5d  |.)?<.%Jxa.v.>..]|
000005c0  7d 6b 0d 75 6b 30 74 0c  26 b2 76 96 1e 60 19 a5  |}k.uk0t.&.v..`..|
000005d0  18 00 00 00 4d 65 61 73  75 72 65 64 20 55 45 46  |....Measured UEF|
000005e0  49 20 6d 65 6d 6f 72 79  20 6d 61 70 11 00 00 00  |I memory map....|
000005f0  04 05 00 00 01 00 00 00  0b 00 00 00 00 00 00 00  |................|
00000600  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00008000
```

The final measurements starting with the description "Measured..." are put in
the log by the Secure Launch kernel code if everything went fine.  During a
poweroff, restart or a kexec of another kernel, the following log lines will
indicate that TXT was properly disabled and SMX mode was left:

```text
[  696.907094] slaunch: TXT clear secrets bit and unlock memory complete.
[  696.914827] slaunch: TXT SEXIT complete.
```

If `tpm2_eventlog` is installed, it can be used parse the log into a readable
form:

```shell
[root@my-system ~]# tpm2_eventlog /sys/kernel/security/slaunch/eventlog
...
```
