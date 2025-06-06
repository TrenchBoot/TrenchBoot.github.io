# Installing TrenchBoot AEM in Qubes OS

This document shows how to install Anti Evil Maid from packages produced by
3mdeb as part of [TrenchBoot as Anti Evil Maid project](https://docs.dasharo.com/projects/trenchboot-aem-v2/).
If you wish to build the components yourself, please refer to documentation for
developers instead.

## Installation

To install, you have to first add a new repository and import a public part of
a key that was used to sign RPM packages.

### Adding AEM repository

To add a new repository, create in dom0 as root `/etc/yum.repos.d/aem.repo`
with the following content:

```text
[aem]
name = Anti Evil Maid based on TrenchBoot
baseurl = https://dl.3mdeb.com/rpm/QubesOS/r4.2/current/dom0/fc37
gpgcheck = 1
gpgkey = https://dl.3mdeb.com/rpm/QubesOS/r4.2/current/dom0/fc37/RPM-GPG-KEY-tb-aem
enabled = 1
```

The key specified in the file must be downloaded and imported to RPM:

```bash
qvm-run --pass-io sys-net \
    'curl -L https://dl.3mdeb.com/rpm/QubesOS/r4.2/current/dom0/fc37/RPM-GPG-KEY-tb-aem' \
    > RPM-GPG-KEY-tb-aem
sudo rpm --import RPM-GPG-KEY-tb-aem
```

Now it should be possible to download and install packages from AEM repository.

### Installing prerequisite packages

As some of the packages are also available in standard QubesOS repositories,
potentially in newer versions, those must be temporarily disabled during
invocation of `qubes-dom0-update`, as shown in the following commands. If any
of the packages that are part of AEM are updated in standard repos, you will
have to choose between using new versions or having working AEM, at least until
new AEM release is published, or the code gets merged upstream. If you decide to
restore AEM after an update broke it, you will have to repeat the installation
of overwritten package with `--action=reinstall` added to `qubes-dom0-update`,
if it wasn’t present before.

#### Qubes repository dependencies

Start by installing prerequisite packages. Those are not part of newly added
repository, but `qubes-dom0-current-testing`:

```bash
sudo qubes-dom0-update --enablerepo=qubes-dom0-current-testing \
    oathtool \
    openssl \
    qrencode \
    tpm-extra \
    trousers-changer \
    tpm-tools
```

#### AEM repository dependencies

Next set of new packages comes from AEM repository, to avoid conflicts other
repositories are disabled for this call:

```bash
sudo qubes-dom0-update --disablerepo="*" --enablerepo=aem \
    grub2-tools-extra
```

##### AMD systems dependencies

This package is only needed on AMD systems:

```bash
sudo qubes-dom0-update --disablerepo="*" --enablerepo=aem \
    secure-kernel-loader
```

#### AEM repository dependencies to reinstall

This is followed by reinstalling additional packages. A reinstallation is required
because currently installed version is equal (or it may be higher in the future)
than those provided by AEM. A couple of GRUB packages differ slightly
depending on whether you use a legacy or UEFI BIOS:

##### Legacy Systems

If your system has a legacy BIOS, reinstall these packages:

```bash
sudo qubes-dom0-update --disablerepo="*" --enablerepo=aem --action=reinstall \
    python3-xen \
    xen \
    xen-hypervisor \
    xen-libs \
    xen-licenses \
    xen-runtime \
    grub2-common \
    grub2-pc \
    grub2-pc-modules \
    grub2-tools \
    grub2-tools-minimal
```

Depending on the set of currently installed packages and their versions
the commands required to install the dependencies vary.

`--action=reinstall` will make sure that the existing packages will be replaced
with the custom versions provided by the AEM repo. That will only happen if
the version number of the installed package matches exactly the version in the
repository.

If any package returns `No match for argument`, then it might need to be
installed, not reinstalled. To make sure that missing packages are installed
re-run the exact command with `--action=install` instead.
`--action=install` won't install packages if the are already installed, even
if the repository contains packages with higher version number.

If a package was skipped during both `--action=reinstall` and `--action=install`
it might need to be updated instead using `--action=update`. This will happen
if the version number in the repository is higher than the currently installed
one. `reinstall` won't work because the versions don't match and `install` won't
work because the package is already installed.

Make sure that all of the packages listed above are installed in the system
and come from the AEM repository.

###### Updating GRUB on legacy systems

Booting on legacy systems requires manual installation of GRUB2 to the MBR
of disk where Qubes OS is stored. If you are sure where the root partition
is located, you can skip the following steps explaining how to find it out.

To check on which drive is your OS installed, run:

1. Get the Logical Volume (LV) file:

    ```bash
    df --output=source /
    ```

    Example LV file: `/dev/mapper/qubes_dom0-root`

1. Map the Logical Volume (LV) to a Volume Group (VG):

    ```bash
    sudo lvs /dev/mapper/qubes_dom0-root --noheadings -o vg_name
    ```

    Example VG name: `qubes_dom0`

1. Map the Volume Group to a Physical Volume (PV):

    ```bash
    sudo vgs --noheadings -o pv_name qubes_dom0
    ```

    Example PV file: `/dev/mapper/luks-12345678-1234-1234-1234-123456789abc` (GUID)

1. Find the PV name in `lsblk` output and check on which device it is
located.

    ```bash
    lsblk -o NAME
    ```

    Example output:

    ```text
    sda
    ├─sda1
    ├─sda2
    └─sda3
      └─luks-12345678-1234-1234-1234-123456789abc
    ```

    The drive on which the root partition is located in this example is
    therefore `/dev/sda`.

Remember that GRUB2 must be installed on disk and
not on partition, so don’t use `sda1`, `nvme0n1p1` etc. This step should be
skipped on UEFI systems.

```bash
sudo grub2-install /dev/sda
```

##### UEFI Systems

If your system has an UEFI BIOS, install these packages instead:

```bash
sudo qubes-dom0-update --disablerepo="*" --enablerepo=aem --action=reinstall \
    python3-xen \
    xen \
    xen-hypervisor \
    xen-libs \
    xen-licenses \
    xen-runtime \
    grub2-common \
    grub2-efi-x64 \
    grub2-efi-x64-modules \
    grub2-tools \
    grub2-tools-minimal
```

Depending on the set of currently installed packages and their versions
the commands required to install the dependencies vary.

`--action=reinstall` will make sure that the existing packages will be replaced
with the custom versions provided by the AEM repo. That will only happen if
the version number of the installed package matches exactly the version in the
repository.

If any package returns `No match for argument`, then it might need to be
installed, not reinstalled. To make sure that missing packages are installed
re-run the exact command with `--action=install` instead.
`--action=install` won't install packages if the are already installed, even
if the repository contains packages with higher version number.

If a package was skipped during both `--action=reinstall` and `--action=install`
it might need to be updated instead using `--action=update`. This will happen
if the version number in the repository is higher than the currently installed
one. `reinstall` won't work because the versions don't match and `install` won't
work because the package is already installed.

Make sure that all of the packages listed above are installed in the system
and come from the AEM repository.

### Installing main AEM package

Finally, `anti-evil-maid` package may be installed:

```bash
sudo qubes-dom0-update --disablerepo="*" --enablerepo=aem anti-evil-maid
```

## Provisioning

All packages are in place. Before we can proceed with provisioning AEM, the TPM
must be cleared in the BIOS. Some platforms may require disabling Intel Trusted
Execution Technology (TXT) in order to clear TPM. After you clear the TPM,
remember to enable Intel TXT back, otherwise AEM will not work. Once TPM is
cleared, perform the TPM setup:

```bash
sudo anti-evil-maid-tpm-setup
```

![](../img/qubes_aem_setup.png)

You will be prompted to set the SRK password, it is a password to access TPM’s
nonvolatile storage where the AEM secrets will be sealed. If you failed to
clear the TPM, you will be shown a message like this:

![](../img/qubes_aem_setup_fail.png)

In that case, try clearing the TPM and run `sudo anti-evil-maid-tpm-setup`
again.

Now all that's left is proper installation of AEM. There are different options,
refer to `anti-evil-maid-install -h` for examples. In the simplest case, AEM is
installed on boot partition (not disk, i.e. `sda1` instead of `sda` etc.) of
Qubes OS. Run this command to find out where your boot partition is installed:

```bash
df --output=source /boot
```

Assuming that your boot partition is installed on `/dev/sda1`, the
installation can be done with a simple command:

```bash
sudo anti-evil-maid-install /dev/sda1
```

![](../img/qubes_aem_install.png)

After that, reboot the platform. On first boot you will be asked for SRK
password, followed by another question for disk encryption password, after which
a screen mentioning absent secret file will be shown:

![](../img/qubes_aem_1st_boot.png)

This is expected on the first boot after installation or an update to one or
more of measured components (GRUB, Xen, dom0 kernel and initramfs).
