# The Assay Parameters table

The ASSAY\_PARAMETERS table has the following structure...

```text
======================  ===================== ========= ========== =======================================================
COLUMN_NAME             Data Type             Nullable  Inserts   Updates     Comments

ASSAY_PARAM_ID          NUMBER(9,0)           No        Numeric Primry Key
ASSAY_ID                NUMBER(9,0)           No
TYPE                    VARCHAR2(200 BYTE)    No
RELATION                VARCHAR2(10 BYTE)     No
VALUE                   NUMBER(9,*)           Yes
TEXT_VALUE              VARCHAR2(200 BYTE)    Yes
UNITS                   VARCHAR2(200 BYTE)    Yes
TYPE_NORMALIZED         VARCHAR2(200 BYTE)    Yes
RELATION_NORMALIZED     VARCHAR2(10 BYTE)     Yes
VALUE_NORMALIZED        NUMBER(9,*)           Yes
TEXT_VALUE_NORMALIZED   VARCHAR2(200 BYTE)    Yes
UNITS_NORMALIZED        VARCHAR2(200 BYTE)    Yes
COMMENTS                VARCHAR2(2000 BYTE)   Yes
ACTIVE                  NUMBER(1,0)           No
TYPE_N_FIXED            NUMBER(1,0)           Yes



A ‘Job’ is defined as the collection of depostion files loaded together in a single run of the loader. A successful deposition results in the assignment of a ‘job_id’ to the entire operation and the data associated with it (see Embargo, Tagging and Freezing section later for more on job_ids). A ‘deposition’ represents the atomic transaction loading event: a ‘deposition’ either completely succeeds to load, or it completely fails.

.. _DDIDs:

Depositor-Defined IDs
=====================
```

The use of identifiers for key entities, created and maintained by the source to be updated is a vital component of a simple, automatable mechanism for maintaining updatable data in ChEMBL. The names \(or 'headers' in a deposition file\) for such identifiers are of the form ‘\*IDX’ where the ‘\*’ may be any capital letter, and ‘X’ refers to the fact that it is externally managed. This format is designed to clearly distinguish this class of identifiers from other \(internal\) ChEMBL identifiers. Currently there are three such \*IDX’s, covering the 3 main entity types in ChEMBL: Assays, Compounds and References. Each of these are defined in a 'primary' deposition file, as show below...

Depositors may use upto 200 characters to define \*IDX’s. Depositors are free to use whatever UniCode characters they wish to use \(but non-visible characters will be stripped from both ends\). All deposited files must be ascii or UTF-8 encoded.

## RIDX

Depositor-defined Reference ID \(RIDX\) is used to define what was previously referred to as a Reference\_ID in deposited literature-bioactivity data files, although here we propose it’s definition is widened to accommodate all forms of deposited data in ChEMBL. We propose that the mapping of a reference\_id \(now RIDx\) to a document\_id be kept, but this therefore requires that the current definition of the document\_id should also be widened: ‘Documents’ may include URLs, or simply text descriptions of the collection of data referred to by the reference.

Also, the mapping of a RIDX to a DOC\_ID must be constrained to a 1:1 relationship. This is enforced by DOC\_ID being the PK on the DOCS table, and by a unique constraint enabled for the “RIDX / SRC\_ID ” combination on this table. This means that a DOC\_ID can never be shared between SRC\_IDs.

Within the LoadType definition for RIDX, constraints would be placed on the required fields to enable the depositor to either cite a literature reference, or populate other fields which would describe the ‘project’ or ‘batch’ of the data set. For example, if the ‘journal’ field is not populated, for one LoadType, then perhaps a ‘Depositor Summary of the Study’ would be required instead, or perhaps an ‘experimental batch\_id’ could be quoted. In this way, the doc\_id may serve several purposes other than its traditional role of simply identifying a ‘literature reference’.

Each CIDX or AIDX defined by a depositor can only be assigned to a SINGLE RIDX, although the same RIDX may be assigned to multiple CIDX’s and AIDX’s. A depositor may only assign RIDX’s defined by themselves, and may only assign them to their own CIDX’s and AIDX’s. A depositor may also assign their own RIDX’s \(but not others’ RIDX’s\) to their own Activity records, even when the activity record uses AIDS and CIDS defined by other depositors.

### Default DOC\_IDs and RIDXs

The new table, as shown, contains the ‘TRUV’ paradigm \(TYPE, RELATION, UNITS, VALUE\) to be consistent with ACTIVITY\_PROPERTIES and ACTIVITY\_SUPP, and to some extent ACTIVITIES.

For consistency with COMPOUND\_RECORDS, a field ‘TYPE\_N\_FIXED’ would also exist to ‘fix’ the TYPE\_NORMALIZED field for the record. Non-null value in here would prevent Normalization step 1 \(see below\) from updating the TYPE\_NORMALIZED field on this record.

An additional field of ‘ACTIVE’ would indicate whether the parameter was currently assigned to the Assay\_ID, or was once in the past \(this prevents us from deleting previous curation efforts if parameters get updated several times\). All records with ‘ACTIVE’ as ‘null’ would be deleted as part of the release process \(in the released schemas, not the production system\). For analysis, joining to between ASSAYS and ASSAY\_PARAMETERS would require a ‘where ACTIVE is not null’ clause, unless the analysis was done on the released data \(where it should be done\).

**Loading**

Loading wrt the ASSAY\_PARAMETERS table would work as follows, in two steps:  
Firstly, if the ASSAY\_PARAM deposition file contained ANY TYPE for a given AIDX, then ALL existing records for the assay\_id \(corresponding to the AIDX\) in the ASSAY\_PARAMETER table would have the ‘ACTIVE’ set to null \(effectively dissociating all these TYPEs from the assay\_ID\).

Secondly, updates or inserts would occur as follows… \[These operations are also summarized on the table diagram above\] New TYPEs for an assay\_id would be inserted as a new record, and Existing TYPEs for an assay\_id would simply update the existing record for this TYPE and assay\_id. Other fields would be inserted / updated as described in the table diagram above.

Normalization.

Normalization \(updating of [\*]()\_NORMALIZATION fields in ASSAY\_PARAMETERS\) would be a 2 step process…

System Message: WARNING/2 \(assayParams.rst, line 117\); [_backlink_]()

 Inline emphasis start-string without end-string.

1.Update TYPE\_NORMALIZED fields. Mappings of TYPES to NORMALIZED\_TYPES \(stored in a ‘ASSAY\_PARAMETER\_TYPES’ table\), would be used to generate TYPE\_NORMALIZED from TYPE in all records in ASSAY\_PROPERTIES, wiping previous values in this field, EXCEPT those records where TYPE\_N\_FIXED is NOT null. Thus, the TYPE\_N\_FIXED field would mean the TYPE\_NORMALIZED field would NOT be changed during this step of the normalization process. TYPES remaining null after this process \(because a record does not exist in ‘ASSAY\_PARAMETER\_TYPES’\) would be flagged for curators to add a new record, then re-run this process. 2.Update VALUES\_NORMALIZED, RELATIONS\_NORMALIZED and UNITS\_NORMALIZED according to rules based on the ‘NORMALIZED\_TYPE’ for a record. This would INCLUDE records where ‘TYPE\_N\_FIXED’ is not null.

