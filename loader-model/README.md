# The Loader Model

Here we describe the model used by the loader.

## Jobs.

A ‘Job’ is defined as the collection of depostion files loaded together in a single run of the loader. A successful deposition results in the assignment of a ‘job\_id’ to the entire operation and the data associated with it \(see Embargo, Tagging and Freezing section later for more on job\_ids\). A ‘deposition’ represents the atomic transaction loading event: a ‘deposition’ either completely succeeds to load, or it completely fails.

## Depositor-Defined IDs

The use of identifiers for key entities, created and maintained by the source to be updated is a vital component of a simple, automatable mechanism for maintaining updatable data in ChEMBL. The names \(or 'headers' in a deposition file\) for such identifiers are of the form ‘\*IDX’ where the ‘\*’ may be any capital letter, and ‘X’ refers to the fact that it is externally managed. This format is designed to clearly distinguish this class of identifiers from other \(internal\) ChEMBL identifiers. Currently there are three such \*IDX’s, covering the 3 main entity types in ChEMBL: Assays, Compounds and References. Each of these are defined in a 'primary' deposition file, as show below...

| [\*]()IDX | Description | Primary File |
| :--- | :--- | :--- |
| AIDX | Depositor-defined Assay ID | ASSAY |
| CIDX | Depositor-defined Compound ID | COMPOUND\_RECORD |
| RIDX | Depositor-defined Reference ID | REFERENCE |

Depositors may use upto 200 characters to define \*IDX’s. Depositors are free to use whatever UniCode characters they wish to use \(but non-visible characters will be stripped from both ends\). All deposited files must be ascii or UTF-8 encoded.

### RIDX

* An RIDX may refer to the results from a given publication, or a single unpublished dataset.
* RIDXs must be unique within a source. Where a source contains submissions from multiple datasets or sites, using a unique RIDX for each one will make it possible to distinguish between them in ChEMBL.
* It is possible to deposit additional data for an existing RIDX.
* ‘Documents’ may include URLs, or simply text descriptions of the collection of data referred to by the reference.
* RIDXs map 1:1 to an internal DOC\_ID field. This is enforced by DOC\_ID being the PK on the DOCS table, and by a unique constraint enabled for the “RIDX / SRC\_ID ” combination on this table. This means that a DOC\_ID can never be shared between SRC\_IDs.

**Relations to compounds and assays**

* Each CIDX or AIDX defined by a depositor can only be assigned to a SINGLE RIDX, although the same RIDX may be assigned to multiple CIDX’s and AIDX’s. 
* A depositor may only assign RIDX’s defined by themselves, and may only assign them to their own CIDX’s and AIDX’s. 
* A depositor may also assign their own RIDX’s \(but not others’ RIDX’s\) to their own Activity records, even when the activity record uses AIDXs and CIDXs defined by other depositors.

#### Default DOC\_IDs and RIDXs

All src\_id's are associated with a single 'default' document\_id. When a new src\_id is created it must be assigned a ‘default’ document\_id. The RIDX associated with this default document\_id is always named 'default'. Th default RIDX and document\_id is always used for incoming data unless a depositor specifies a RIDX during loading. The ‘default’ doc\_id definition, can, of course, be manually edited by the ChEMBL administrator in consultation with the depositor.

For a given ‘Deposition’, a depositor may decide to NOT associate one of their RIDX’s to one of their CIDX’s, AIDX’s or Activities. If they do not, then the loader will automatically assign the ‘default’ RIDX for this SRC\_ID.

The DOCS table requires that a unique constraint of a ‘SRC\_ID / RIDX’ combination be enforced. Because multi-field unique constraints in Oracle cannot accept ‘null’ in any field, the string ‘default’ is used for the RIDx field when no RIDx has been specified. This means that the depositor must be aware that the string ‘default’ is reserved as the RIDx for their default RIDx. Attempting to create or edit a RIDx called ‘default’ will therefore be met with an error by the loader, although its use to confirm that the Activity data set should be assigned to ‘default’ \(by putting ‘default’ in the RIDx field in the ACTIVITY file\) is be permitted \(although leaving blank, or not specifying the field at all would have the same effect\).

### CIDX

Depositor-defined Compound IDs \(CIDXs\) are defined in COMPOUND\_RECORD deposition files. Currently this file has only one mandatory field: CIDx. In other words, a depositor may simply deposit a series of CIDx values and nothing else \(no structures, compound\_name, compound\_key, nothing\)\] Compounds with structures must have their structures defined separately in the secondary depostion file: COMPOUND\_CTAB. All CIDx identifiers in secondary files \(incl the sdf file\) must either exist within the COMPOUND.txt in the same deposition set, or have been deposited in a previous deposition set \(in a COMPOUND\_RECORD file\) by the same depositor.

### AIDX

Depositor-defined ASSAY IDs \(AIDXs\) are defined in ASSAY deposition files.

## Deposition files.

Currently permitted Deposition Files are given on the 'Requirements' page.

Deposition files are classed as either ‘‘Primary’, ‘Secondary’ or 'Tertiary'. ‘Primary’ files contain and define the primary keys for a particular Depositor-defined entity \(eg: AIDX for ASSAYS\). ‘Secondary’ files are keyed on the primary keys from these primary files \(eg: In the secondary file ‘ASSAY\_PARAM’, AIDX is a FK to the PK in ASSAY\). Teriary files do not define [\*]()IDX's in any way, but serve to define relationships between them in some way \(such as ACTIVITY and all other related files\).

Secondary files serve to further define [\*]()IDX's where...

* multiple records may be required to define multiple definitions for a single \*IDX \[eg: multiple parameters for a single AIDX\), or,
* the format of the primary file cannot hold the data associated with the [\*]()IDX \(eg: a CTAB structure in an SDF \)

  System Message: WARNING/2 \(loaderModel.rst, line 82\);

  Inline emphasis start-string without end-string.

### Loading Logic for Deposition Files

During the loading process Primary and secondary files are treated slightly differently. Considering each of these categories separately…

#### Primary Files

Primary files update and create the major entities in ChEMBL. Thus, the 3 primary files have the following effects at load time…

ASSAY files define AIDX’s. If the AIDX does not yet exist in the ASSAYS table for this src\_id then one is created. However, if one already exists, then the old one is overwritten with the new one.

REFERENCE files define RIDX’s. If the RIDX does not yet exist in the DOCS table for this src\_id then one is created. However, if one already exists, then the old one is overwritten with the new one.

COMPOUND\_RECORD files define combinations of CIDX and RIDX. If a CIDX-RIDX combination does not yet exist in the COMPOUND\_RECORDS table for this src\_id, then one will be created. If one already exists, then this existing one will be updated \(with the COMPOUND\_NAME, etc, present in the incoming file\).

#### Secondary Files

Secondary files \(ie: ASSAY\_PARAM and COMPOUND\_CTAB\) cannot result in the creation of the entities referred to above, but serve to hang extra data off these entities. The Depositor-defined IDs used in these files must already be present, either in the DB or in corresponding primary file in the same deposition. Apart from this dependency, secondary files nevertheless work relatively independently of the primary files. Thus, the presence of \*IDX data in secondary files will wipe and replace the secondary data associated with these \*IDXs, regardless of whether the corresponding \*IDX is present in the corresponding primary file. Similarly, the absence of a \*IDX from a secondary file will have no effect on these associated data, and to delete associated secondary data then empty records for the \*IDX must be present in the secondary files. This behaviour is made clearer by some examples for each of the two secondary files…

**ASSAY\_PARAM**

Suppose that in a single deposition the ASSAY\_PARAM file contains multiple records for the AIDX ‘abc’. If ‘abc’ already exists in the DB \(or in the ASSAY file in the same deposition\), then these parameters will be loaded against the ‘abc’ assay, replacing all previous parameters loaded against ’abc’. However, if the same file contains a single record for AIDX ‘def’ with empty values in ‘TYPE’ and ‘VALUE’ columns, then all existing parameters associated with ‘def’ will be deleted from the DB, and no new ones created. Suppose also that in the same deposition the ASSAY file defines or redefines AIDX ‘ghi’, but that no records exist for ‘ghi’ in the ASSAY\_PARAM file. In this case, if ‘ghi’ does not already exist, then it will be created with no parameters. However, if it does already exists, then it will be updated, but any existing secondary parameters associated with ‘ghi’ in the ASSAY\_PARAMETER table will not be affected. This behaviour prevents the inadvertent deletion of secondary data when an assay is updated.

**COMPOUND\_CTAB**

Suppose that in a single deposition the COMPOUND\_CTAB file contains a record for CIDX ‘str1’, and that the CTAB for this defines a structure.. If ‘str1’ already exists in the DB \(or in the COMPOUND\_RECORD file in the same deposition\), then the old CTAB for ‘str1’ will be replaced with the new structure, and ALL records in the COMPOUND\_RECORDS table which contain ‘str1’ will be automatically assigned a new molregno for this new structure. However, if the CTAB for ‘str1’ is empty, then ALL records in the COMPOUND\_RECORDS table which contain ‘str1’ will be automatically assigned a new \[blank?\] molregno \(with no structure assigned to it in ChEMBL\). Similarly, if the COMPOUND\_CTAB file does not contain ‘str1’ at all, but the COMPOUND\_RECORDS file does, then the records for ‘str1’ in the COMPOUND\_RECORD table will be updated \(eg: COMPOUND\_NAME, etc\), but the structure will not be updated \(the molregno for the record will not be changed\).

Thus, the only way to update the structure for CIDX ‘str1’ from ‘a structure’ \(deposited previously\) to ‘no structure’, is to deposit a COMPOUND\_CTAB file with a record containing a CIDX field ‘str1’, and an empty CTAB.

Thus the absence of data from a secondary file does not imply that the depositor wishes to delete the secondary file data for a \*IDX from the DB.

This loader behaviour is designed to minimize the risk of inadvertently deleting data.

#### Tertiary Files

Files which in some way define relationships between primary identifiers are termed tertiary files \(ACTIVITY, ACTIVITY\_PARAMETERS, ACTIVITY\_SUPP, etc\) A requirement for such files is that referential integrity must be maintained wrt the three \*IDXs that are required to define an activity record: AIDX, CIDX and RIDX. Updating of tertiary files can only be acheived byt a wipe and replace process. The process requires that all ACTIVITY data assigned to a particular JOB\_ID is wiped \(but not primary and secondary file data for the same JOB\_ID\) and the updated ACTIVITY data then re-loaded. The complexities of loading ACTIVITY and more complex data is dealt with [here](../complex-results.md).

## Loading against other Depositor’s Entities.

Depositors may load Activity data against the Depositor–defined IDs managed by other depositors. To permit this, Activity deposition files make use of headers which flag this information to the loader. Typically, these headers have the form ‘SRC\_ID\_\*IDX’, where ‘\*’ may be an ‘A’ or ‘C’, and contain the src\_id \(integer\) corresponding to the depositor who owns the \*IDX referred to in the same record. More details of this are given later in this document.

When loading against other depositors AIDX’s or CIDX’s, Activity records created in the ‘ACTIVITIES’ table in ChEMBL will reference the ASSAY\_ID and the RECORD\_ID \(resp\) of the AIDX and CIDX defined by the other depositor.

Clearly, a depositor can only define and redefine their own Depositor defined IDs, and clearly, a depositor can only use another source’s \*IDX if it is already defined in ChEMBL. Although allowing the use of \*IDX’s from other depositors creates great flexibility in the data model, depositors should be made very aware of the potential dangers of doing this, since a \*IDX used in this way cannot be edited by them, and indeed may be edited by the real owner at any time, without notifying anyone. Depositing against other depositors \*IDX’s should therefore ONLY be done in cases where the depositor is very confident in the ability of the other depositor to maintain their \*IDX’s correctly.

## Deposition \(DEP\_\) Tables

In order to capture all deposited data, deposition files are ‘mirrored’ in ChEMBL with an equivalent table \(prefixed with ‘DEP\_’\) which contains all permitted fields for each deposition file. Note that DEP\_ tables do NOT exist to capture the entire history of previous versions of an entity. Rather, they exist ONLY to hold the current definitions of the depositors entity. They therefore only serve as a means only of supporting the current content of ChEMBL.

DEP\_ tables for ‘Entity-defining’ files are managed straightforwardly, as they are wiped and replaced as entities are defined and redefined. For these tables only the addition of a ‘src\_id’ field is required \(and an ‘STDINCHIKEY’ field for DEP\_COMPOUND\_CTAB\). However, the DEP\_ACTIVITY table is slightly more complicated. For DEP\_ACTIVITY an additional unique ‘Activity\_id’ is added at load time, to tie each record to the corresponding record in the ACTIVITY table. The ‘Activity Deletion’ steps \(described in BioAssays below\) would require the removal of records in DEP\_ACTIVITY with the same activity\_id as the deleted records. The inclusion of job\_id on the DEP\_ACTIVITY would probably also assist with some types of deletions.

DEP\_ tables must never be manually edited.

