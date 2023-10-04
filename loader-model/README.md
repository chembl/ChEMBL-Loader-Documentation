# The Loader Model

Here we describe the model used by the loader.

## Jobs.

A ‘Job’ is defined as the collection of deposition files loaded together in a single run of the loader. A successful deposition results in the assignment of a ‘job\_id’ to the entire operation and the data associated with it (see Embargo, Tagging and Freezing section later for more on job\_ids). A ‘deposition’ represents the atomic transaction loading event: a ‘deposition’ either completely succeeds to load, or it completely fails.



## Loading against other Depositor’s Entities.

Depositors may load Activity data against the Depositor–defined IDs managed by other depositors. To permit this, Activity deposition files make use of headers which flag this information to the loader. Typically, these headers have the form ‘SRC\_ID\_\*IDX’, where ‘\*’ may be an ‘A’ or ‘C’, and contain the src\_id (integer) corresponding to the depositor who owns the \*IDX referred to in the same record. More details of this are given later in this document.

When loading against other depositors AIDX’s or CIDX’s, Activity records created in the ‘ACTIVITIES’ table in ChEMBL will reference the ASSAY\_ID and the RECORD\_ID (resp) of the AIDX and CIDX defined by the other depositor.

Clearly, a depositor can only define and redefine their own Depositor defined IDs, and clearly, a depositor can only use another source’s \*IDX if it is already defined in ChEMBL. Although allowing the use of \*IDX’s from other depositors creates great flexibility in the data model, depositors should be made very aware of the potential dangers of doing this, since a \*IDX used in this way cannot be edited by them, and indeed may be edited by the real owner at any time, without notifying anyone. Depositing against other depositors \*IDX’s should therefore ONLY be done in cases where the depositor is very confident in the ability of the other depositor to maintain their \*IDX’s correctly.

## Deposition (DEP\_) Tables

In order to capture all deposited data, deposition files are ‘mirrored’ in ChEMBL with an equivalent table (prefixed with ‘DEP\_’) which contains all permitted fields for each deposition file. Note that DEP\_ tables do NOT exist to capture the entire history of previous versions of an entity. Rather, they exist ONLY to hold the current definitions of the depositors entity. They therefore only serve as a means only of supporting the current content of ChEMBL.

DEP\_ tables for ‘Entity-defining’ files are managed straightforwardly, as they are wiped and replaced as entities are defined and redefined. For these tables only the addition of a ‘src\_id’ field is required (and an ‘STDINCHIKEY’ field for DEP\_COMPOUND\_CTAB). However, the DEP\_ACTIVITY table is slightly more complicated. For DEP\_ACTIVITY an additional unique ‘Activity\_id’ is added at load time, to tie each record to the corresponding record in the ACTIVITY table. The ‘Activity Deletion’ steps (described in BioAssays below) would require the removal of records in DEP\_ACTIVITY with the same activity\_id as the deleted records. The inclusion of job\_id on the DEP\_ACTIVITY would probably also assist with some types of deletions.

DEP\_ tables must never be manually edited.
