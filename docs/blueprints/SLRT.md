# Secure Launch Resource Table

## Summary

SLRT is a data format whose purpose is to describe Late Launch related
information and pass it from a bootloader to DCE and then to DLME.  As SLRT is
being processed it can even be updated in place.

The format is defined by [Secure Launch Specification][slaunch-spec].

[slaunch-spec]: ../specifications/Secure_Launch.md

## Background

Intel TXT and AMD SKINIT follow very different approaches: while TXT has a
dedicated register space and extensive data structures exposing necessary
information to DCE and DLME, SKINIT is a bare mechanism which leaves all the
details to an implementation.  This leaves an implementation that
wants to support both DRTM solutions with two main choices:

* handle each kind of DRTM in its own specific way
* come up with a way to mostly unify different DRTMs and build code around that

SLRT is the result of choosing the latter.

## Approach

SLR table is stored sequentially in physical memory and can be described with an
address pointing at its start.

Fields are naturally aligned up to 64 bits (so 128-bit data would be aligned to
64 bits as well).

Most of the entries and their fields are generic enough to apply to multiple
current and possibly some future implementations, although some fields might not
be strictly necessary in some cases.

Platform- and architecture-specific data are contained in dedicated entries,
so that only relevant information needs to be specified in any given
environment.

The SLRT is prepared by a bootloader and then passed down to DLE and DLME in a
way which depends on a given platform.
