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

* Depositor-defined Compound IDs \(CIDXs\) are defined in COMPOUND\_RECORD deposition files. 
* The CIDX string should be menaingful to the depositor, for example the identifier, name or abbreviations used in the publication. 
* Currently this file has only one mandatory field: CIDx.  A depositor may simply deposit a series of CIDx values and nothing else.
* Compounds with structures must have their structures defined separately in the secondary depostion file: COMPOUND\_CTAB. 
* All CIDx identifiers in secondary files \(incl the sdf file\) must either exist within the COMPOUND.txt in the same deposition set, or have been deposited in a previous deposition set \(in a COMPOUND\_RECORD file\) by the same depositor.

### AIDX

* Depositor-defined ASSAY IDs \(AIDXs\) are defined in ASSAY deposition files.
* Should be a string that is meaningful to the depositor, and must be unique within the given source.
* If your dataset has multiple collaborators, giving each collaborator a prefix for their AIDXs will prevent duplicate AIDXs and aid in tracking who submitted what.
* An assay record records information on the protocol used for the experiment. 
* Each assay can only have a single target.
* In the case of a panel assay, there will need to be one Assay record per target.
* Standard reference assays that are used repeatedly can be deposited as a specific dataset, with just that one assay/group/panel.

## Loading against other Depositor’s Entities.

Depositors may load Activity data against the Depositor–defined IDs managed by other depositors. To permit this, Activity deposition files make use of headers which flag this information to the loader. Typically, these headers have the form ‘SRC\_ID\_\*IDX’, where ‘\*’ may be an ‘A’ or ‘C’, and contain the src\_id \(integer\) corresponding to the depositor who owns the \*IDX referred to in the same record. More details of this are given later in this document.

When loading against other depositors AIDX’s or CIDX’s, Activity records created in the ‘ACTIVITIES’ table in ChEMBL will reference the ASSAY\_ID and the RECORD\_ID \(resp\) of the AIDX and CIDX defined by the other depositor.

Clearly, a depositor can only define and redefine their own Depositor defined IDs, and clearly, a depositor can only use another source’s \*IDX if it is already defined in ChEMBL. Although allowing the use of \*IDX’s from other depositors creates great flexibility in the data model, depositors should be made very aware of the potential dangers of doing this, since a \*IDX used in this way cannot be edited by them, and indeed may be edited by the real owner at any time, without notifying anyone. Depositing against other depositors \*IDX’s should therefore ONLY be done in cases where the depositor is very confident in the ability of the other depositor to maintain their \*IDX’s correctly.

## Deposition \(DEP\_\) Tables

In order to capture all deposited data, deposition files are ‘mirrored’ in ChEMBL with an equivalent table \(prefixed with ‘DEP\_’\) which contains all permitted fields for each deposition file. Note that DEP\_ tables do NOT exist to capture the entire history of previous versions of an entity. Rather, they exist ONLY to hold the current definitions of the depositors entity. They therefore only serve as a means only of supporting the current content of ChEMBL.

DEP\_ tables for ‘Entity-defining’ files are managed straightforwardly, as they are wiped and replaced as entities are defined and redefined. For these tables only the addition of a ‘src\_id’ field is required \(and an ‘STDINCHIKEY’ field for DEP\_COMPOUND\_CTAB\). However, the DEP\_ACTIVITY table is slightly more complicated. For DEP\_ACTIVITY an additional unique ‘Activity\_id’ is added at load time, to tie each record to the corresponding record in the ACTIVITY table. The ‘Activity Deletion’ steps \(described in BioAssays below\) would require the removal of records in DEP\_ACTIVITY with the same activity\_id as the deleted records. The inclusion of job\_id on the DEP\_ACTIVITY would probably also assist with some types of deletions.

DEP\_ tables must never be manually edited.

