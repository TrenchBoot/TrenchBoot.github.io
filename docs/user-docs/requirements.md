# TrenchBoot hardware requirements

TrenchBoot by itself doesn't have any special requirements, just use
the requirements of target OS or hypervisor as a base. However, the
same can't be said about vendors DRTM implementation, and those differ
between vendors.

## Intel

For Intel, you need a TXT-capable chipset and TXT-capable CPU. In
addition, a TPM is required. It shouldn't make a difference whether it
is a discrete chip (dTPM) or firmware TPM (fTPM) running in a CPU's
trusted execution environment, but version (1.2 or 2.0) must be
supported by the processor, or more specifically, the SINIT
Authenticated Code Module (ACM) for given CPU family. Refer to
[Late Launch Overview](../dev-docs/Late_Launch_Overview.md) for more
information about the role and source of ACM.

Depending on firmware, TXT may need to be manually enabled in setup.

## AMD

On AMD, DRTM implementation is tightly connected to Secure Virtual
Machine (SVM) extensions. This is a virtualization technology included
in all recent (last decade or even longer) CPUs. Contrary to Intel, a
discrete TPM (dTPM) is required on AMD, but its version doesn't matter
\- the counterpart to ACM (which must be signed by Intel) is
implemented by TrenchBoot with support for both TPM 1.2 and 2.0.

Depending on firmware, SVM (which may be labeled also as
virtualization or AMD-V) may need to be manually enabled in setup.

## Caveats

While the above information summarizes requirements provided by
respective vendors, it doesn't necessarily mean that the platform
meeting those conditions will fully work. It is possible that the
firmware doesn't initialize all of the parts required by DRTM, or ACM
is buggy, or there is an issue with microcode that doesn't properly
start a dynamic launch event. A list of such quirks can be found
[here](test_matrix.md/#hardware-quirks-and-workarounds).
