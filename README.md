# shrub

<img src="assets/logo.png" alt="shrub logo" height="88">
 
**shrub** is a best-effort DICOM de-identification tool implementing the
**Basic Application Level Confidentiality Profile** defined in DICOM
PS3.15 Annex E, along with a selection of its standard Options.
 
> ⚠️ **Not a guarantee of anonymity.** shrub follows the DICOM
> de-identification standard mechanically, but the standard itself notes
> that using a confidentiality profile is only *part* of a de-identification
> process — it does not replace a broader process of determining context,
> interpreting applicable regulations (HIPAA, GDPR, etc.), and assessing
> re-identification risk. Always review output before release.
 
## What it does
 
For each processed DICOM file, shrub:
 
- Applies the Basic Application Level Confidentiality Profile action codes
  (`D`, `Z`, `X`, `K`, `C`, `U`, ...) to every Attribute listed in Table
  E.1-1 of PS3.15, whether in the top-level Data Set or nested inside
  Sequence Items.
- Sets **Patient Identity Removed (0012,0062)** to `YES`.
- Populates **De-identification Method (0012,0063)** and/or
  **De-identification Method Code Sequence (0012,0064)** with the
  Profile/Option codes applied (per CID 7050).
- Replaces the **SOP Instance UID** and all references to other SOP
  Instances that could be used to trace back to the patient.
- Strips or replaces **File Meta Information**, including the 128-byte
  preamble, so identity can't leak through unmodified headers.
- Removes any unrecognized **Group 0004** (DICOMDIR) elements from
  non-DICOMDIR files.
## Supported Options
 
Options are applied on top of the Basic Profile to control how much
clinically-relevant context is retained:
 
| Option | Behavior |
|---|---|
| Retain Safe Private | Keeps private Attributes believed not to contain identifying info |
| Retain UIDs | Preserves original UIDs instead of replacing them |
| Retain Device Identity | Keeps device/equipment identification Attributes |
| Retain Institution Identity | Keeps institution name/address Attributes |
| Retain Patient Characteristics | Keeps age, sex, size, weight, etc. |
| Retain Longitudinal Temporal Info | Preserves date/time offsets for longitudinal studies |
| Clean Descriptors | Scrubs free-text Attributes (e.g. Study Description) for embedded identity info |
| Clean Structured Content | Scrubs identity info from SR/structured content |
| Clean Graphics | Scrubs burned-in or overlay graphic annotations |
 
Unless an Option is explicitly enabled, shrub falls back to the Basic
Profile's default action for each Attribute.
 
## Limitations
 
- shrub cannot detect identity information burned into **pixel data**
  (e.g. text overlays in ultrasound/secondary captures) unless paired
  with an OCR/redaction step — this is outside the scope of PS3.15 Annex E.
- Private Attributes not covered by a "known safe" list are removed by
  default; false positives/negatives are possible.
- Encrypted Attributes Data Set (re-identification) support is limited —
  check the conformance statement before relying on recovery.
## Conformance statement
 
Per PS3.15 §E.1.3, shrub's actual behavior — which Attributes are
removed, which are replaced with dummy values (and how), which are kept
for later re-identification, and its scope of referential integrity for
UIDs — is documented in `CONFORMANCE.md` (generate this per deployment
before relying on shrub for regulated data).
 
## Status
 
Best-effort, community-maintained. Not certified. Use at your own risk
for clinical, research, or regulatory purposes.