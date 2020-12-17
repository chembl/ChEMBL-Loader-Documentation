# Compound Normalization

In the loader, the normal operation for loading data and normalizing cpds is to run these two operations in 2 separate stages \(using different utilities of the same loader script\). 

* The first stage \('Loading a Job'\) is run manually during the day. Multiple 'Loading a Job' runs may be done in one day. 
* The second stage \('Normalization'\) is kicked off nightly with cron.

 The second of these two stages is the focus of this document, although some information on the first stage is also included, as this is required for a full understanding of the second stage. Before describing these processes, however, it is useful to describe some general aspects of the data model that are directly relevant to compound normalization.

## Normalization aspects of the data model.

A full description of the data model is explained elsewhere  Here, are some details of the model relevant to the Normalization procedure.

A job, from a given depositor 'S', may include either one, or both, or neither, of the files: 

A Job may include compound data. If it does, these data are supplied in the files:

* COMPOUND\_RECORD.sdf. This file is loaded to the DEP\_COMPOUND\_RECORD table.
* COMPOUND\_CTAB.sdf. This file is loaded to the DEP\_COMPOUND\_CTAB table.

### COMPOUND\_RECORD

Records in COMPOUND\_RECORD.txt are keyed on a combination of CIDX and RIDX \(composite primary key\). Each record in this file either creates a new record in COMOUND\_RECORDS table \('CR'\), or, if the CIDX/RIDX combination for this depositor already exists in COMPOUND\_RECORDS, then updates the existing CR record.

### COMPOUND\_CTAB

Records in the COMPOUND\_CTAB.sdf file are keyed on CIDX. A job will fail if a COMPOUND\_CTAB.sdf contains CIDX's which are not in the accompanying COMPOUND\_RECORD.txt file \(if it exists\) or in the DB already. All deposited CTABs in the COMPOUND\_CTAB.sdf are run through the InChI generation software, and the InChI, if successfully created, is stored in the DEP\_COMPOUND\_CTAB file.

Three new fields have been added to CR for use by the new loader: MOLREGNO\_FIXED, MOLREGNO\_COMMENTS and MOLREGNO\_SV. These are required for the normalization process.

**MOLREGNO\_FIXED**

A '1' in MOLREGNO\_FIXED indicates the MOLREGNO, MOLREGNO\_FIXED and MOLREGNO\_COMMENTS fields must NOT be changed by any automatic normalization process. A '1' in this field typically arises from a curator who has decided that the molregno for this record should be assigned manually, perhaps because they know that the structure referred to has actually been drawn incorrectly by the depositor. Records with a 'null' in MOLREGNO\_FIXED, may have the MOLREGNO and MOLREGNO\_COMMENTS fields updated automtically by the normalization process described below.

**MOLREGNO\_COMMENTS**

The MOLREGNO\_COMMENTS field is a free text field, and is used to explain why a particular normalization has occurred. If a curator has entered a '1' in the MOLREGNO\_FIXED field, then they are encouraged to also populate the MOLREGNO\_COMMENTS field with a reason for their manual assignment. If a curator subsequently decides that their manual normalization is incorrect, then they can EITHER update the MOLREGNO, and the MOLREGNO\_COMMENTS, but leaving the MOLREGNO\_FIXED field as '1', OR, set all these three fields to null and let the automatic process populate these fields overnight.

The 'Standardization' process refers to the process of re-formatting the CTAB into a consistent appearance, and correcting errors of chemistry representation where possible. The 'Standardization' process occurs as part of the Normalization process via a web service. 

#### MOLREGNO\_SV

The MOLREGNO\_SV field is available to capture the Version of the Standardizer software used returned by the web-service along with the data for each structure queried, as detailed below in rules 2 and 3.

## Stage 1: Loading a job \(util1 in 'cloader.py'\)

This involves setting certain cr.MOLREGNO fields to 'null', depending upon what has been loaded in the job, as described below. This stage also involves an assessment of the quality of each deposited structure using a 'structure checker' web service. 

This service returns a list of comments on the quality of each structure, and a 'Penalty Score' for each \(see [Running the Loader](https://app.gitbook.com/@chembl/s/chembl-loader/~/drafts/-LvpPM2s7ephYXEXdJAX/running-loader#rules-alerts-warnings-and-errors) for more info on Penalty Scores\). These scores are not used during stage 1, but are stored in the DEP\_COMPOUND\_CTAB\_LOG table for later use in 'Stage 2' \(further below\). No other Standardization/Normalization' steps are carried out at this stage.

#### CIDX/RDX 

* If a COMPOUND\_RECORD.txt contains a **new** CIDX/RIDX combination, a new record is loaded to CR, with cr.MOLREGNO set to 'null', and cr.MOLREGNO\_COMMENT set to 'new record'
* If a COMPOUND\_RECORD.txt contains a CIDX/RIDX **update**, the relevant fields are updated \(eg: COMPOUND\_KEY, etc\), but none of the cr.MOLREGNO\* fields are updated.

#### InChI Keys

* For new CIDXs in COMPOUND\_CTAB.sdf, if a valid InChIKey is not created, then do nothing. 
* If the deposited CTAB generated a valid InChIKey, then:
  *  For ALL CR records containing CIDX for 'S' \(regardless of the RIDX in the record\) the cr.MOLREGNO is set to 'null' \(and cr.MOLREGNO\_COMMENT set to 'new structure'\), 
  * Those records which also have a '1' in the MOLREGNO\_FIXED field are left unchanged.
* For CIDXs which already contain a record in the DEP\_COMPOUND\_CTAB table, If the InChIKey for the new structure is different to the InChIKey for the previously deposited structure for this CIDX then:
  * For ALL CR records containing CIDX for 'S' \(regardless of the RIDX in the record\) the cr.MOLREGNO is updated to 'null'.
  * cr.MOLREGNO\_COMMENT is updated to 'updated structure'\), 
  * This update applies when an InChI Key has been updated, removed, or added.
  * Records where cr.MOLREGNO\_FIXED is not null are ignored by this update. 
  * If the InChIKey has not changed, nothing will be updated.

For all the above possibilities, MOLREGNO\_SV is always set, or updated to, 'null'.

## Stage 2 Structure Normalization \(util10 in 'cloader.py'\)

### Introduction.

'Normalization' refers to the process of assigning an appropriate molregno to a CR record. 'Standardization' refers to the process of taking a CTAB and applying a number of rules to it, and returning a redrawn, 'cleaner' CTAB. 

The aim is to produce more consistently drawn structures in the DB and correct some errors of chemistry. The standardization process uses a web-service-based protocol. This service accepts a CTAB and returns a standardized form of the CTAB. The 'version' of the standardization protocol used behind this web-service is also available from this web call and is used to populate the cr.MOLREGNO\_SV field, as detailed below in rules 2 and 3.

Each normalization process that is started is assigned a record in the NORMALIZATION\_CPD table. During the normalization process, an number of operations are considered worthy of 'flagging' to curators, such as normalizations where some CR records have 'fixed' molregnos for the same CIDX. These are written to the NORMALIZATION\_CPD\_LOG table \(with FKs to the PK of the NORMALIZATION\_CPD table\) after every batch commit \(see 'Batch operation of the Normalization Process' below\). For more information on these logs see 'Logging normalizations for curator verification.' below.

### The Process.

* All normalization steps apply only to records where molregno is null and JOB\_ID &gt;0. This ensures that data that have already been processed, and data that were loaded with the old loader are ignored. 
* Retrieve a non-redundant list of all CIDX's \(for each SRC\_ID\) _from_ COMPOUND\_RECORDS.
* For each CIDX, apply the following rules, in order, until a molregno is assigned. Once a molregno is assigned, carry out the 'Action' required by the rule, and then proceed to the next CIDX. 

Note that the update statements generated for all rules below will update ALL records for the CIDX's to the molregno selected by the rule \(regardless of the RIDX, or whether cr.molregno is null or not\), and will include the where clause 'where cr.MOLREGNO\_FIXED is null and JOB\_ID &gt; '0'.'

Note also that CIDX/SRC\_ID combinations which have one or more records with a Penalty Score of 6 or more in the DEP\_COMPOUND\_CTAB\_LOG table \(as generated in stage 1, above\) are excluded from rules 2 and 3. This is to prevent any structures with particular problems, identified during loading by the structure checker, from being loaded into ChEMBL. These CIDX/SRC\_ID combinations are assigned a molregno only on the basis of rules 1, 4 and 5.

For several rules below, the MOLREGNO\_COMMENT field may be updated with text to describe the rule employed. 

* In these cases, in addition to the text described below, the text 'insert' or 'match' is also be appended. 
* This describes whether the molregno assignment in this case was achieved by 'matching' to an existing iKey within ChEMBL, or whether no such iKey currently exists, and so a new molregno, and iKey, was 'inserted' into ChEMBL. T
* his information is required for the 'scanForCleanerStructures' process which may be run after normalization. See "[Scanning for cleaner structures"](scan-clean.md) for more information on this**.**

#### Rule 1

**Has this CIDX already been automatically normalized to a molregno ?**

This rule involves identifying an existing single CR record for this CIDX and source, where cr.MOLREGNO is not null, and cr.MOLREGNO\_FIXED is null. This represents a molregno which will have been assigned automatically to the CIDX on a previous run of the normalizer, and which can therefore be used again. Compound normalization fields \(eg: MOLREGNO, MOLREGNO\_SV, etc\) can be 'copied' from this single record.

In the event that there is more than one CR record where cr.MOLREGNO\_FIXED is null for this CIDX, then use the following 'single CR-selection process' to select just one CR record from which to copy values from...

* First, select the CR record\(s\) with the most recent MOLREGNO\_SV.
* Of these, select record\(s\) with the most commonly occurring molregno.
* Of these, select the record\(s\) with lower molregno value.
* Of these, select the record with the highest record\_id.

**Action**: Update MOLREGNO, also setting MOLREGNO\_SV and MOLREGNO\_COMMENT to the values found in the single selected CR record.

NB: At this stage, in the same DB query, it is also useful to identify if there are ANY CR records where MOLREGNO\_FIXED is '1', and then tag this CIDX as 'previously manually fixed'. In other words, a CIDX tagged in this way indicates that there is at least 1 CR record for this CIDX where the MOLREGNO\_FIXED is '1'. 

This information is, in itself, of no use for assigning a molregno if the CIDX passes this rule, but may be useful for future reference in subsequent rule testing \(see below\), and is also important for the generation of the 'NORMALIZATION\_CPD\_LOG', which is designed to log normalization steps which may require verification by curators \(see later below\).

#### Rule 2

**Does a deposited structure, which can be ‘standardized’, exist for this CIDX ?**

For rule 2, a CIDX is assessed according to the following 3 criteria, each tested in turn…

* The CIDX has a deposited structure \(ie: a record in DEP\_COMPOUND\_CTAB\) which is not associated with a penalty score of 6 or more in the DEP\_COMPOUND\_CTAB\_LOG table.
* The ‘standardization web service’ executes successfully when queried with the deposited CTAB for this CDIX. ‘Success’ of the ‘standardization web service’ occurs if the following are ALL True:
  1. The return code is '200'.
  2. The UPDATEDCTAB key exists.
  3. The VERSION key exists and the associated value is a number.
* The value associated with the UPDATEDCTAB key from the ‘standardization web service’ can subsequently be used successfully to obtain a valid standard InChI using the InChI generation software.

The CIDX passes rule 2 only if ALL three criteria of the above are True. If, at any stage, a criterion is not satisfied, then the rule fails immediately, and no further criteria are assessed for that CIDX.

Note, therefore, that the ‘standardization web service’ may completely succeed \(ie: pass criteria a, b, and c\), but if the subsequent InChI generation of the retrieved CTAB fails \(step 3\) then the CIDX has failed rule 2. This may occur, for example, if the ‘CTAB’ associated with the CIDX is an empty string, or a string which is not a valid CTAB 

The expected behavior of the ‘standardization web service' is for a, b and c to all succeed whatever the service is queried with\) However, also note that this circumstance is unlikely to occur, as such a ‘structure’ is unlikely to have passed criterion ‘1’.

**Action**: Update MOLREGNO, then MOLREGNO\_SV is set to the version number of the ‘standardization web service’ \(2c above\), and the MOLREGNO\_COMMENT is set to **‘standardized structure’.**

#### Rule 3

**Does a deposited structure, which can generate a standard InChI, exist for this CIDX ?**

For rule 3, a CIDX is assessed according to the following 2 criteria, tested in turn…

* The CIDX has a deposited structure \(ie: a record in DEP\_COMPOUND\_CTAB\) which is not associated with a penalty score of 6 or more in the DEP\_COMPOUND\_CTAB\_LOG table.
* The CTAB from the above record can be used successfully to obtain a valid standard InChI using the InChI generation software.

The CIDX passes rule 3 only if BOTH of the above criteria are True.

**Action**: Update MOLREGNO, MOLREGNO\_COMMENT is set to **‘non-standardized structure’**. MOLREGNO\_SV is set to the ‘version’ number obtained if this CIDX was assessed by rule 2.

Note that all CIDXs that are assessed by rule 3 will have previously been assessed by rule 2 \[and so sent to the ‘standardization web service’, as per step 2 of the previous rule\]. If rule 3 has been passed, then it is informative to capture the version number of the ‘standardization web service’ that was used when testing this CIDX with rule 2 \(if it was captured successfully\), and to populate the MOLREGNO\_SV with this value. 

In this way, the version of the standardization protocol that failed to process the structure to a valid, standardized, CTAB is captured.

If, when rule 2 failed, the version was not successfully captured, then MOLREGNO\_SV is set to ‘0’. Note therefore, that for this rule, a MOLREGNO\_SV of ‘0’ means that: “The structure was sent to the ‘standardization web service’ during an assessment of rule2, but rule 2 failed. One of the reasons for failing was that the version number could not be retrieved from the data returned from the ‘standardization web service’ call.”. 

The corollary of this is that a MOLREGNO\_SV &gt; 0 means: “The structure was sent to the ‘standardization web service’, and this version of the standardization protocol was run, but rule 2 failed. Rule 2 failed either because the standardization web service failed \[according to either criteria a or b above\], or the retrieved CTAB failed to generate a valid InChI”.

#### Rule 4

**Does a 'previously manually fixed' molregno exist for this CIDX ?**

The data for this will have been retrieved from the DB during rule 1 testing \(see above\). In the event that there is more than one other CR record where cr.MOLREGNO\_FIXED is not null for this CIDX, then use the 'single CR-selection process' described above to identify a single record.

**Action**: Update MOLREGNO, also setting MOLREGNO\_SV to 'null' and MOLREGNO\_COMMENT to 'copy of fixed structure'.

#### Rule 5

**Does a deposited structure exist for this CIDX ?**

In this case, the ‘deposited structure’ will have already failed rules 2 and 3. Note that the definition of a ‘deposited structure’ is that a record exists for this CIDX in DEP\_COMPOUND\_CTAB. The CTAB in this record may contain any string, including an empty one.

If a CIDX passes rule 5, then a new ‘empty structure’ molregno is created, with a structure type of ‘NONE’ in the Molecule\_Dictionary, and the MOLREGNO field assigned this value.

**Action**: Update MOLREGNO, also setting MOLREGNO\_SV to ‘null’ and MOLREGNO\_COMMENT to **‘invalid structure’**

#### Rule 6

**No structure has been deposited for this CIDX.**

No CTAB has been deposited for this CIDX \(ie: No record exists for this CIDX in the DEP\_COMPOUND\_CTAB table\). In this case, a new ‘empty structure’ molregno is created, with a structure type of ‘NONE’ in the Molecule\_Dictionary, and the MOLREGNO field assigned this value.

**Action**: Update MOLREGNO, also setting MOLREGNO\_SV to ‘null’ and MOLREGNO\_COMMENT to **‘no structure’**.

**Checking for existing Molregnos**

* If rules 2,3,4 or 5 apply, then use the InChI key to look for an existing molregno \(getMolregnoForStructure\). 
* If no molregno exists, create a new molregno and structure in ChEMBL
* If the InChI Key is empty, or represents an empty molfile, then create a NEW molregno representing an EMPTY structure

### Some observations on this process.

#### Logging noteworthy normalization events for curator verification.

During the Normalization process, some automated events or steps are considered worthy of being 'flagged' for the manual attention of curators. Such events which may warrant manual edits to normalizations of the records affected, or to other processes.

Currently, the following messages may be generated in the NORMALIZATION\_CPD\_LOG table...

* "MOLREGNO updated" \# ie: mrn1 -&gt; mrn2, or mrn1 -&gt; null \(but NOT null -&gt; mrn1\)
* "MOLREGNO normalized to a FIXED molregno" \# ie: 'a rule 4 normalization'
* "A fixed molregno exists in another record\_id" \# The CIDX/SRC\_ID was normalized to a particular MOLREGNO which is different to the MOLREGNO assigned to the same CIDX/SRC\_ID in another record.
* "MOLREGNO FIXED but molregno is null"
* "Standardization changed iKey" \# ie: The standardization process resulted in the inChIKey being changed. Developers of the Standardization process are interested to know if this process significantly alters structure.

These log messages are timestamped and FK'ed to the record\_ids which are affected, and FK'ed to a NORMALIZATION\_CPD table which records details of each individual run of the Normalizer.

#### Possibility of molregnos changing for empty structures.

* If, over a series of jobs, a depositor sends multiple updates for the same CIDX, all of which are invalid CTABs, then the same 'empty' molregno will stay assigned to the CIDX. 
* If a CIDX has no structure when first deposited, subsequently the depositor provides a valid structure, and then on a third deposition thet provide an invalid structure, then the original molregno cannot be 'reused' for the third deposition. 
* This can result in the molregno for a given CIDX changing over time, although this scenario is considered rare.

#### Applying a new 'version' of the standardization protocol \(util12 in 'cloader.py'\)

Periodically, the 'web standardization protocol' may be updated \(The protocol is 'versioned' to indicate that changes have been made\). Occassionally, these updates to the standardization protocol may be large enough to warrant the re-normalization of all \('new-loader'\) ChEMBL data using this new version.

This is achieved by running utility number 12. 

* This util first captures a list of all CIDX's \(and SRC\_IDs\) with MOLREGNO\_SV's lower then the current version.
  * It excludes CR records where MOLREGNO\_FIXED is not null, and uses the normal JOB\_ID filter to apply only to new loader data. 
* Starting with the 'oldest' MOLREGNO\_SV, each CIDX is run through rules 1 and 2 above, but \(for rule 1\) copying only from records where MOLREGNO\_SV is equal to the current version of the standardizer. 
* If, using rule 2, the new version of the standardizer fails to standardize a CIDX that had previously been standardized using an earlier version of the standardizer, then no updates are carried out for this CIDX, but this failure is logged to the curator's log.

#### Batch operation of the Normalization Process.

The normal operation for the 'Compound Normalization' utility in the loader is to:

* Run a DB query to identify all CIDX/SRC\_ID combinations requiring normalization.  
* Process these in batches of 10 through a normalization process \(described in detail by util11\)
* Commit at the end of each batch. 

The commit step requires a response from the user, although if the '-a' option is used then the default is used instead \('yes, commit' in this case\). Processing of batches will continue until all CIDX/SRC\_IDs have been normalized.

_Command-line options_

* If the user wishes to change the batch size from 10, then they can do this with the '-y' \(lower case\) option \(which takes an integer\). 
* If the user wishes to limit the total number of CIDX/SRC\_IDs processed in one run of the script, then this may be altered in a similar way by using the '-Y' option \(uppercase\). 
* For example, running with the options ' -y 20 -Y 502 ' will result in the processing of 502 CIDX/SRC\_IDs, 20 at a time \(ie: 26 batches, with 20 CIDX/SRC\_IDs in all but the last batch, which would contain 2 CIDX/SRC\_IDs\).

The standardization process included in rule 2 involves the calling of a ‘standardization web service’. 

* This service is designed to always ‘succeed’ \(according to the three criteria, a, b and c in Rule 2 above\), even if the query CTAB is an empty string, or an invalid CTAB. 
* If the service should fail, then CIDX’s which should be ‘standardized’ will be marked as ‘non-standardized’, with a molregno\_SV set to ‘0’. 
* Failure of the service may be due to either 
  * 1. The attempted processing of a string which it cannot handle \[hopefully unlikely\]. 
  * 2. The service becoming unavailable \(web server failure, network problems, etc\). 
* Currently, the entire normalization process will abort if two successive queries fail. This behavior is intended to prevent the inadvertent assignment of incorrectly ‘non-standardized’ records in the event of a service failure for reason 2, but not reason 1. 
* Aborting the process results in the program exiting with non-zero exit status, which can be used as a signal to the user that a problem has occurred. 
* Setting -Y and -y to reasonably low numbers \(and perhaps stringing together multiple script runs consecutively on the LSF\) mitigates the risk of network interruptions resulting in the 'loss' of large numbers of normalizations before they can be committed. 
* However, the cost of setting -Y and -y too low, is that: 
  * 1. Each run of the Normalization process requires a full query of the CR table to assess which CIDX/SRC\_IDs currently require normalization, and 
  * 2. Each batch requires a single run of the InChI generation software \(it is far more efficient to run this for multiple structures simultaneously.\)

#### Additional Processes of interest.

After a normalization process has been completed, it is sometimes useful to run a process to identify whether any 'cleaner' structures were deposited recently. This process is described [here](scan-clean.md).

