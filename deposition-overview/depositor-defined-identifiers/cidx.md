---
description: Depositor-defined Compound IDs
---

# CIDX

* Depositor-defined Compound IDs (CIDXs) are defined in COMPOUND\_RECORD deposition files.&#x20;
* The CIDX string should be meaningful to the depositor, for example the identifier, name or abbreviations used in the publication.&#x20;
* Currently this file has only one mandatory field: CIDx.  A depositor may simply deposit a series of CIDx values and nothing else.
* Compounds with structures must have their structures defined separately in the secondary deposition file: COMPOUND\_CTAB.&#x20;
* All CIDx identifiers in secondary files (incl the sdf file) must either exist within the COMPOUND.txt in the same deposition set, or have been deposited in a previous deposition set (in a COMPOUND\_RECORD file) by the same depositor.
