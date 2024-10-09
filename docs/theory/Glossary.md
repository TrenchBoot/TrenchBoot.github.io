# Glossary

Provided are definitions of terms used throughout TrenchBoot's documents and
designs to encourage a common vocabulary and understanding.

## AMD SKINIT

AMD's implementation of Dynamic Launch. "AMD Secure Startup with SKINIT" is the
longer version.

## Authenticated Code Module (ACM)

A proprietary software component built and signed by Intel that is used as a DCE
for Intel TXT. Also referred to as "SINIT", "SINIT ACM" or "SINIT AC module".

## Dynamic Configuration Environment (DCE)

A software component like ACM or Secure Loader that is the first to run in a
secure execution environment before passing control to DLME.

## Dynamic Launch (DL)

A system launch that can be done repeatedly with the execution code able to
reside at different locations in memory. This is sometimes referred to as a
"Late Launch". This is related to dynamic RTM (D-RTM, D-CRTM, DRTM).

## Dynamic Launch Event (DLE)

A special instruction of a processor supporting Dynamic Launch that initiates
the process (e.g., `GETSEC[SENTER]` on Intel or `SKINIT` on AMD) resulting in
the execution of DCE.

## Dynamic Launch Measured Environment (DLME)

A software component (e.g., an OS or a hypervisor) that gets started by DCE.
Known as MLE in case of Intel TXT.

## Explicit Trust

When a trustor has explicitly established a degree of trust with a trustee.

## Implicit Trust

When a trustor has relied upon a trustee to establish a degree of trust with
another trustee.

## Intel Trusted Execution Technology (Intel TXT)

Intel's implementation of Dynamic Launch.

## Measured Launched Environment (MLE)

A term for DLME as used by Intel TXT.

## Root of Trust (RoT)

An idempotent mechanism whereby the result is used to assert a fact about the
entity it acted upon.

## Root of Trust for Measurement (RTM, CRTM)

The first entity in the chain of measurements. Also called "Core RTM". Can be
static (SRTM) or dynamic (DRTM).

## Secure Kernel Loader (SKL)

Free and open source implementation of a Secure Loader for AMD Secure Startup
with SKINIT.

## Secure Launch (SL, slaunch)

A specification describing a common approach for implementing Dynamic Launch.

## Secure Launch Resource Table (SLRT)

A platform-agnostic, standard format for providing information to the DLE
Handler and for passing relevant information across the DLE to be available for
use by the DCE and the DLME.

## Static Launch

A system launch that is a one time execution with the execution code at a fixed
location in memory. This is related to static RTM (S-RTM, S-CRTM, SRTM).

## Secure Loader (SL)

A user-provided binary which acts as a DCE for AMD SKINIT.

## Secure Loader Block (SLB)

A region of memory into which Secure Loader is loaded.

## Transitive Trust

An operation conducted by a trustor that consists of one or more mechanisms
used to assess one or more facts about a trustee before allowing the trustee to
be included within the trustor's trust boundary and delegated the authority to
act as a trustor.

## Trust

Assured reliance on the properties, ability, strength, or truth of an entity.

## Trust Anchor

The result of a Root of Trust mechanism that is a fact being relied upon to assert
correctness, e.g. trustworthiness.

## Trust Boundary

A demarcation that identifies a subset of entities as those that a trustor has
explicitly or implicitly established as trustworthy.

## Trustee

An entity that is trusted by another entity.

## Trustor

An entity that establishes a degree of trust of another entity.
