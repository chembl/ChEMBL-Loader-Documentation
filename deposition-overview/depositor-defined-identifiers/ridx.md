---
description: Depositor-defined Reference IDs
---

# RIDX

* An RIDX may refer to the results from a given publication, or a single unpublished dataset.
* Should be a string that is meaningful to the depositor, and must be unique within the given source.
* RIDXs must be unique within a source. Where a source contains submissions from multiple datasets or sites, using a unique RIDX for each one will make it possible to distinguish between them in ChEMBL.
* It is possible to deposit additional data for an existing RIDX.
* ‘Documents’ may include URLs, or simply text descriptions of the collection of data referred to by the reference.
* RIDXs map 1:1 to an internal DOC\_ID field. This is enforced by DOC\_ID being the PK on the DOCS table, and by a unique constraint enabled for the “RIDX / SRC\_ID ” combination on this table. This means that a DOC\_ID can never be shared between SRC\_IDs.

**Relations to compounds and assays**

* Each CIDX or AIDX defined by a depositor can only be assigned to a SINGLE RIDX, although the same RIDX may be assigned to multiple CIDX’s and AIDX’s.&#x20;
* A depositor may only assign RIDX’s defined by themselves, and may only assign them to their own CIDX’s and AIDX’s.&#x20;
* A depositor may also assign their own RIDX’s (but not others’ RIDX’s) to their own Activity records, even when the activity record uses AIDXs and CIDXs defined by other depositors.

#### Default DOC\_IDs and RIDXs

All src\_id's are associated with a single 'default' document\_id. When a new src\_id is created it must be assigned a ‘default’ document\_id. The RIDX associated with this default document\_id is always named 'default'. Th default RIDX and document\_id is always used for incoming data unless a depositor specifies a RIDX during loading. The ‘default’ doc\_id definition, can, of course, be manually edited by the ChEMBL administrator in consultation with the depositor.

For a given ‘Deposition’, a depositor may decide to NOT associate one of their RIDX’s to one of their CIDX’s, AIDX’s or Activities. If they do not, then the loader will automatically assign the ‘default’ RIDX for this SRC\_ID.

The DOCS table requires that a unique constraint of a ‘SRC\_ID / RIDX’ combination be enforced. Because multi-field unique constraints in Oracle cannot accept ‘null’ in any field, the string ‘default’ is used for the RIDx field when no RIDx has been specified. This means that the depositor must be aware that the string ‘default’ is reserved as the RIDx for their default RIDx. Attempting to create or edit a RIDx called ‘default’ will therefore be met with an error by the loader, although its use to confirm that the Activity data set should be assigned to ‘default’ (by putting ‘default’ in the RIDx field in the ACTIVITY file) is be permitted (although leaving blank, or not specifying the field at all would have the same effect).
