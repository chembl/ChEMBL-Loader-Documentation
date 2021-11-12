# Loader files

## Deposition files.

Currently permitted Deposition Files are given on the 'Requirements' page.

Deposition files are classed as either ‘‘Primary’, ‘Secondary’ or 'Tertiary'. ‘Primary’ files contain and define the primary keys for a particular Depositor-defined entity (eg: AIDX for ASSAYS). ‘Secondary’ files are keyed on the primary keys from these primary files (eg: In the secondary file ‘ASSAY\_PARAM’, AIDX is a FK to the PK in ASSAY). Teriary files do not define [\*](broken-reference)IDX's in any way, but serve to define relationships between them in some way (such as ACTIVITY and all other related files).

Secondary files serve to further define [\*](broken-reference)IDX's where...

* multiple records may be required to define multiple definitions for a single \*IDX \[eg: multiple parameters for a single AIDX), or,
*   the format of the primary file cannot hold the data associated with the [\*](broken-reference)IDX (eg: a CTAB structure in an SDF )

    System Message: WARNING/2 (loaderModel.rst, line 82);

    Inline emphasis start-string without end-string.

### Loading Logic for Deposition Files

During the loading process Primary and secondary files are treated slightly differently. Considering each of these categories separately…

#### Primary Files

Primary files update and create the major entities in ChEMBL. Thus, the 3 primary files have the following effects at load time…

ASSAY files define AIDX’s. If the AIDX does not yet exist in the ASSAYS table for this src\_id then one is created. However, if one already exists, then the old one is overwritten with the new one.

REFERENCE files define RIDX’s. If the RIDX does not yet exist in the DOCS table for this src\_id then one is created. However, if one already exists, then the old one is overwritten with the new one.

COMPOUND\_RECORD files define combinations of CIDX and RIDX. If a CIDX-RIDX combination does not yet exist in the COMPOUND\_RECORDS table for this src\_id, then one will be created. If one already exists, then this existing one will be updated (with the COMPOUND\_NAME, etc, present in the incoming file).

#### Secondary Files

Secondary files (ie: ASSAY\_PARAM and COMPOUND\_CTAB) cannot result in the creation of the entities referred to above, but serve to hang extra data off these entities. The Depositor-defined IDs used in these files must already be present, either in the DB or in corresponding primary file in the same deposition. Apart from this dependency, secondary files nevertheless work relatively independently of the primary files. Thus, the presence of \*IDX data in secondary files will wipe and replace the secondary data associated with these \*IDXs, regardless of whether the corresponding \*IDX is present in the corresponding primary file. Similarly, the absence of a \*IDX from a secondary file will have no effect on these associated data, and to delete associated secondary data then empty records for the \*IDX must be present in the secondary files. This behaviour is made clearer by some examples for each of the two secondary files…

**ASSAY\_PARAM**

Suppose that in a single deposition the ASSAY\_PARAM file contains multiple records for the AIDX ‘abc’. If ‘abc’ already exists in the DB (or in the ASSAY file in the same deposition), then these parameters will be loaded against the ‘abc’ assay, replacing all previous parameters loaded against ’abc’. However, if the same file contains a single record for AIDX ‘def’ with empty values in ‘TYPE’ and ‘VALUE’ columns, then all existing parameters associated with ‘def’ will be deleted from the DB, and no new ones created. Suppose also that in the same deposition the ASSAY file defines or redefines AIDX ‘ghi’, but that no records exist for ‘ghi’ in the ASSAY\_PARAM file. In this case, if ‘ghi’ does not already exist, then it will be created with no parameters. However, if it does already exists, then it will be updated, but any existing secondary parameters associated with ‘ghi’ in the ASSAY\_PARAMETER table will not be affected. This behaviour prevents the inadvertent deletion of secondary data when an assay is updated.

**COMPOUND\_CTAB**

Suppose that in a single deposition the COMPOUND\_CTAB file contains a record for CIDX ‘str1’, and that the CTAB for this defines a structure.. If ‘str1’ already exists in the DB (or in the COMPOUND\_RECORD file in the same deposition), then the old CTAB for ‘str1’ will be replaced with the new structure, and ALL records in the COMPOUND\_RECORDS table which contain ‘str1’ will be automatically assigned a new molregno for this new structure. However, if the CTAB for ‘str1’ is empty, then ALL records in the COMPOUND\_RECORDS table which contain ‘str1’ will be automatically assigned a new \[blank?] molregno (with no structure assigned to it in ChEMBL). Similarly, if the COMPOUND\_CTAB file does not contain ‘str1’ at all, but the COMPOUND\_RECORDS file does, then the records for ‘str1’ in the COMPOUND\_RECORD table will be updated (eg: COMPOUND\_NAME, etc), but the structure will not be updated (the molregno for the record will not be changed).

Thus, the only way to update the structure for CIDX ‘str1’ from ‘a structure’ (deposited previously) to ‘no structure’, is to deposit a COMPOUND\_CTAB file with a record containing a CIDX field ‘str1’, and an empty CTAB.

Thus the absence of data from a secondary file does not imply that the depositor wishes to delete the secondary file data for a \*IDX from the DB.

This loader behaviour is designed to minimize the risk of inadvertently deleting data.

#### Tertiary Files

Files which in some way define relationships between primary identifiers are termed tertiary files (ACTIVITY, ACTIVITY\_PARAMETERS, ACTIVITY\_SUPP, etc) A requirement for such files is that referential integrity must be maintained wrt the three \*IDXs that are required to define an activity record: AIDX, CIDX and RIDX. Updating of tertiary files can only be acheived byt a wipe and replace process. The process requires that all ACTIVITY data assigned to a particular JOB\_ID is wiped (but not primary and secondary file data for the same JOB\_ID) and the updated ACTIVITY data then re-loaded. The complexities of loading ACTIVITY and more complex data is dealt with [here](../design-specification-and-internal-docs/complex-results.md).

##
