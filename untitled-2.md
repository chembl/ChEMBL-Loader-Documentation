# Glossary

## Depositor-Defined ID.

      An identifier for a key entity. There are 3 kinds of Depositor-Defined IDs

* CIDX   ... A depositor-defined Compound ID
* AIDX   ... A depositor-defined Assay ID
* RIDX   ... A depositor-defined Reference ID

The format of such identifiers is always '\*IDX', where \* is always an uppercase letter.

* Such identifiers are always generated and maintained by the depositor themselves.
* If the definition of the identifier in ChEMBL needs to be updated by the depositor, then the same identifier must be used in another deposition, and the original definition is then overwritten.
* Identifiers must consist of a sequence \(between 1 and 200\) of any visible ascii characters \(non-visible characters will be stripped from both ends\).

## Deposition Job.

* A ‘Deposition Job’ is defined as the collection of files loaded together in a single run of the loader.
* A successful deposition results in the assignment of a ‘job\_id’ to the entire operation and the data associated with it \(see Embargo, Tagging and Freezing section later for more on job\_ids\).
* A deposition job represents the atomic transaction loading event: The loading of a deposition job either completely succeeds to load, or it completely fails.

## LoadType.

 When loading, the ChEMBL administrator may have a particular understanding or expectation of the result of loading a particular job, based on their knowledge of the nature and load history of the source.

Thus, for example, some sources might be considered to usually only provide data where the load would not be expected to update any entities previously deposited in the DB \(but to simply incrementally add more entities\), and for the referential integrity to be fully defined within the job itself \(ie: with no reference to previously deposited entities\).

An example of such a source would be molconn literature data, where each load is usually from a new collection of research papers that have not been loaded before, and where 'updates' to previously deposited data are rare.

* For LoadTypes such as these \(called here 'Cumulative Load with job-defined referential integrity'\), the -L option should be set to '2'. \[If -L is not defined, when running the loader, then L defaults to '1'\].
* Changing the '-L' option results in the loader altering the weighting assigned to certain types of warning and errors, in order to highlight and emphasize deviations from the differently anticipated result of the load.

Thus, with -L set to '2'::

1.  If any imminent 'Overwrites' to existing DB entities are detected, then a fatal error is assigned \(these would only be minor alerts with '-L' set to '1'\). Note: an exception to this rule is COMPOUND\_CTAB.sdf \(see below\)\].
2. No 'advanced' checks for better altenative CR CRIDX combinations are carried out \(routinely done for -L = 1, but computationally expensive for large sets\).
3. If there are CIDXs in COMPOUND\_CTAB.sdf which are not present in COMPOUND\_RECORD, then simply ignor them in subsequent processing \[ie: do not allow them to overwite any existing CIDX structures that may exist\], AND demote the errors normally produced in these circumstances \(ie: if the CIDX does not exist in the DB\), to a minor alert. Currently, only 2 LoadTypes are recognized, but others may be added in future, if required.

