# Building and installing Linux

If your system isn't setup for development, you might want to run build commands
inside of [trenchboot-sdk] Docker container which can be started like this:

```bash
docker run --rm -it -v "$PWD:$PWD" -w "$PWD" --user "$(id -u):$(id -g)" \
           -e HOME="$PWD/home" \
           ghcr.io/trenchboot/trenchboot-sdk:master /bin/bash
```

Setting `$HOME` is necessary because `ccache` fails if there is no `$HOME`.

[trenchboot-sdk]: https://github.com/TrenchBoot/trenchboot-sdk

## Preparing for build

```bash
# clone the latest version (at the time of writing)
git clone --branch linux-sl-master-9-12-24-v11 --depth 1 \
          https://github.com/TrenchBoot/linux.git

# change working directory
cd linux

# prepare for out of tree build for simpler cleanup and ability to build
# multiple configs
mkdir tb
export KBUILD_OUTPUT=$PWD/tb
```

## Note on initrd

If initrd is necessary, one might embed required drivers to perform the boot
as part of the configuration or build initrd image after installing everything
by specifying `6.11-rc7-v11-tb` version.  Details on how to perform either of
these options are outside the scope of these instructions.

## Configuring the kernel

Details on what should be enabled and why can be found in
[`Documentation/security/launch-integrity/secure_launch_details.rst`][secure_launch_details],
steps below provide only basic information on configuration options.

Default configuration file is used for a base as an example while you might
want to start with the one from the currently running kernel (`zcat
/proc/config.gz > .config` or `cp /boot/config .config`) and enable the listed
options in a menu (`make menuconfig`) after looking them up via search (hit `/`
and enter, for example, `X86_X2APIC`, then hit `1` to navigate to the option and
change its value).

Start with default configuration file:

```bash
cp arch/x86/configs/x86_64_defconfig "$KBUILD_OUTPUT/.config"
```

Enable X2APIC which is required by TXT:

```bash
echo CONFIG_X86_X2APIC=y >> "$KBUILD_OUTPUT/.config"
```

Disable KASLR which can compromise security or cause crashes (alternatively,
pass `nokaslr` kernel parameter):

```bash
echo '# CONFIG_RANDOMIZE_BASE is not set' >> "$KBUILD_OUTPUT/.config"
```

Select strict IOMMU translated mode by default for better device isolation at
the cost of performance (can be omitted or enabled via `iommu.strict=1` kernel
parameter):

```bash
echo CONFIG_IOMMU_DEFAULT_DMA_STRICT=y >> "$KBUILD_OUTPUT/.config"
```

Enable at least one driver for a TPM without which no DRTM can be done:

```bash
# this might be enough (e.g., for firmware TPM in recent CPUs)
echo CONFIG_TCG_TPM=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_CRB=y >> "$KBUILD_OUTPUT/.config"

# this enables the rest of TPM drivers
echo CONFIG_TCG_TIS=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_I2C=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_I2C_CR50=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_I2C_ATMEL=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_I2C_INFINEON=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_I2C_NUVOTON=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_NSC=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_ATMEL=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_INFINEON=y >> "$KBUILD_OUTPUT/.config"
echo CONFIG_TCG_TIS_ST33ZP24_I2C=y >> "$KBUILD_OUTPUT/.config"
```

TPM drivers are located in
`Device Drivers → Character devices → TPM Hardware Support` menu where more
detailed information about the above options can be found.

Enable Secure Launch itself:

```bash
echo CONFIG_SECURE_LAUNCH=y >> "$KBUILD_OUTPUT/.config"
```

Add a suffix to the version to avoid conflicts, also disable appending of commit
hash:

```bash
echo 'CONFIG_LOCALVERSION="-v11-tb"' >> "$KBUILD_OUTPUT/.config"
echo CONFIG_LOCALVERSION_AUTO=n >> "$KBUILD_OUTPUT/.config"
```

Now make the configuration usable by the build system by completing it:

```bash
make olddefconfig
```

[secure_launch_details]: https://github.com/TrenchBoot/linux/blob/linux-sl-master-9-12-24-v11/Documentation/security/launch-integrity/secure_launch_details.rst

## Build

```bash
# the kernel
make -j$(nproc) bzImage

# and its modules
make -j$(nproc) modules
```

## Installation

The following steps should be run outside of a container either as root user or
with `sudo` prepended to them:

```bash
# kernel
cp "$KBUILD_OUTPUT/arch/x86/boot/bzImage" /boot/vmlinuz-6.11-rc7-v11-tb
cp "$KBUILD_OUTPUT/.config" /boot/config-6.11-rc7-v11-tb
cp "$KBUILD_OUTPUT/System.map" /boot/System.map-6.11-rc7-v11-tb

# modules
make modules_install
```

## Use with TrenchBoot GRUB2

```grub
menuentry 'Linux with Secure Launch 6.11-rc7 v11' --unrestricted {
    insmod part_gpt
    search --no-floppy --fs-uuid --set=root BOOT-FSUUID
    slaunch
    slaunch_module /DCE-FOR-A-GIVEN-PLATFORM
    linux /vmlinuz-6.11-rc7-v11-tb root=PARTUUID=ROOT-PARTUUID ro console=ttyS0,115200n8 console=tty0

    # uncomment if used
    # initrd /initrd-6.11-rc7-v11-tb.img
}
```

Things that must be replaced (add additional kernel parameters and GRUB
commands as needed):

- `DCE-FOR-A-GIVEN-PLATFORM` with path to [ACM] or [SKL] file on boot partition
- `BOOT-FSUUID` with _file-system UUID_ (can be found via `lsblk -o +UUID`)
- `ROOT-PARTUUID` with _partition UUID_ (can be found via `blkid`)
- if `/boot` is not a separate partition, paths should be modified to add
  `/boot` prefix

[ACM]: ../theory/Glossary.md#authenticated-code-module-acm
[SKL]: ../blueprints/AMD_Secure_Kernel_Loader.md
