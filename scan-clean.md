# Scan For Cleaner Structures

This page describes a process which HAS NOT YET BEEN IMPLEMENTED ... but is planned. Please treat information below as 'requirements'.

A given molecule may be drawn \(as a CTAB\) in many ways, and yet still yield the same standard InChIkey \(iKey\). The different possible CTAB representations for the same iKey can include those with errors, inaccuracies and ambiguities.

ChEMBL's repository of structures is the COMPOUND\_STRUCTURES table, where a single CTAB, and a single unique iKey, exists in each record.

Clearly, an ambition of ongoing ChEMBL curation efforts is to optimize the overall quality of each of the single CTABs stored against each iKey.

Occassionally, higher quality CTABs for a given iKey may be deposited from an external source.

Loading and Normalization processes do not identify and highlight such CTABs.

The process described on this page, does identify these 'better' structures, alerts curators to their appearance, and thereby potentially provide a simple way of improving the quality of a ChEMBL CTAB with minimal curation effort in this particular case.

ChEMBL uses a 'structure checker' web service \[link here\] to assess the 'quality' of a CTAB, assigning '[Penalty Scores](https://app.gitbook.com/@chembl/s/chembl-loader/~/drafts/-LvpPM2s7ephYXEXdJAX/running-loader#rules-alerts-warnings-and-errors)' for such undesirable features. ChEMBL uses a 'structure checker' web service \[link here\] to assess the 'quality' of a CTAB, assigning 'Penalty Scores' for features that are judged to be erroneous, inaccurate or ambiguous.

The normalization process \([:ref:\`compound-norm\`](untitled-12.md)\) relies on comparing iKeys of the incoming structure to those iKeys already in ChEMBL. Thus, if the iKey for the incoming structure is not in ChEMBL, then a new molregno is **created** in ChEMBL with this iKey, using the deposited CTAB. On the other hand, however, if the incoming structure iKey **matches** an iKey that is already in ChEMBL, then the molregno for this existing iKey is assigned to the CIDX/SRC\_ID.

A shortcoming of this process is that in the latter \(‘matching’\) situation, no assessment is made as to whether the incoming CTAB might be a better \(or 'cleaner'\) representation of the molecule, lacking the undesirable features mentioned above.The process below,\(fewer Penalty Scores associated with it\) than the existing representation assigned to this iKey in ChEMBL.

If such a CTAB is ‘cleaner’, then it is necessary to bring this matter to the attention of a curator, who may decide that ChEMBL should adopt this new CTAB and discard the old ChEMBL CTAB for this iKey. The purpose of this documentation page is to describe the process \(‘scanForCleanerStructures’\) that is used to identify these potentially cleaner structures, and bring them to the attention of the curator.

## Process: scanForCleanerStructures

This process \(util14\) can be run at anytime, but is most conveniently run after the weekly normalization process is run. The purpose of the process is to identify if there are any recently deposited structures \(CTABs\), which appear to be 'cleaner' than the structure for the same iKey in ChEMBL. The 'cleanliness' of a CTAB is assessed on the basis of the results of the 'structure checker' web service \[link here\], which assigns penalty scores \[[:ref:\`penaltyScores\`]()\] to a CTAB: a lower overall score indicates a 'cleaner' CTAB.

Different running options exist for the process, such as: Run on... 'last N jobs', or 'job\_id X', or 'job\_ids X-Y' \(a range\), or 'all jobs since date'. All CIDX/SRC\_IDs that have had new/updated CTABS deposited in the specified timeframe are identified, and the deposited structure and the corresponding ChEMBL structure \(with the same iKey\) are then both run through the ‘structure checker’ web service. Penalty scores for both are compared, and a report is generated \(example below\), and the data logged to a DB table, highlighting any pairs where the deposited structure appears to be ‘cleaner’ than the existing ChEMBL structure.

All ‘structure checker’ tests are run 'on the fly' using the 'structure checker' web service. This avoids any complications over generating, storing, and updating penalty scores for ChEMBL structures, and problems of scores calculated with different versions of the 'structure checker'. However, the multiple calls to the web service that this requires, means that in order for the process to run in a timely fashion, it is important to only compare relevant pairs of structures, and filter out all pairs where no such comparison is useful. Such 'relevant' pairs are defined as CIDX/SRC\_IDs where either 1. The structure was updated but the iKey did not change \(although the MD5 checksum for the CTAB did change\), or, 2. The CIDX/SRC\_ID was normalized by 'matching' the iKey from the deposited structure to an existing ChEMBL iKey.

Other useful means of reducing web-service calls . . .

1. Run 'structure checker' on the ChEMBL structure, and proceed further ONLY if a penalty score greater than ‘0’ is returned \(obviously, the structure cannot be improved upon if it already has no penalty scores associated with it\)
2. Run each molregno ONLY ONCE, and store result, so if it occurrs again in another pair, then you can re-use the result.

At the end of the process, a report is generated \(as per below\), showing how numbers of CIDX/SRC\_IDs are filtered down to identify relevant records, which are logged to a DB table...

| 123456 |  |  |  |  | Total number of new or updated CIDX/SRC\_ID CTABS |
| :--- | :--- | :--- | :--- | :--- | :--- |
| of these.. |  |  |  |  |  |
|  | 1060 |  |  |  | relevant CIDX/SRC\_IDs according to filter criteria |
|  | of these.. |  |  |  |  |
|  |  | 892 |  |  | ChEMBL structures with a '0' penalty score |
|  |  | 168 |  |  | ChEMBL structures with a &gt; '0' penalty score |
|  |  | of these... |  |  |  |
|  |  |  | 132 |  | SAME penalty score profile for ChEMBL and deposited structure |
|  |  |  | 30 |  | deposited CTAB has a HIGHER penalty score profile than the ChEMBL structure |
|  |  |  | 6 |  | deposited CTAB has a LOWER penalty score profile than the ChEMBL structure |
|  |  |  | of these.. |  |  |
|  |  |  |  | 2 | Are shared but not FIXED. These will be logged |
|  |  |  |  | 1 | Are FIXED but not shared. These will be logged |
|  |  |  |  | 1 | Are shared and FIXED. These will be logged |
|  |  |  |  | 2 | Neither FIXED or shared. \[ ref X \] |

ref X : The last option, will be.. 'Do you wish to just log these, or iterate through now and decide for each one if the CTAB should be updated in ChEMBL ?'

## Some DB/code features required to implement this process.

### 1. A means of identifying all structure updates run for the specified time frame / job\_ids.

The process requires that all CIDX/SRC\_IDs that have had new/updated CTABS deposited in the specified timeframe are first identified. This is achieved by querying the 'UJOB\_ID' field on the DEP\_COMPOUND\_CTAB table.

This field is set to the current JOB\_ID when the record is first inserted, and is updated to the new JOB\_ID whenever the DEP\_COMPOUND\_CTAB record is updated.

Note that the meaning of this field is therefore DIFFERENT to the meaning of all other JOB\_ID fields elsewhere \[where the field indicates the JOB\_ID when the record was first INSERTED\] – embargoing relies on this.

The naming of this field aims to reflect this difference in meaning.

### 2. A means of identifying which CIDX/SRC\_IDs \(either newly added or updated\) had iKeys already present in ChEMBL when they were normalized.

As mentioned above, during the normalization process, if an iKey for an incoming structure is not in ChEMBL, then a new molregno is inserted in the MOLECULE\_DICTIONARY in ChEMBL with this iKey, and using the deposited CTAB. On the other hand, if the iKey matches an iKey that is already in ChEMBL, then the molregno for this existing iKey is assigned to the CIDX/SRC\_ID. The MOLREGNO\_COMMENT created will capture whether this assignement has occurred as a result of an 'insert' or a 'match', by appending these words to the comment assigned according to the rule used \[link here\]

In this way, the scanForCleanerStructures process is able to identify all CIDX/SRC\_IDs in CR where normalization has occurred by ‘matching’ the new structure iKey to an already existing iKey in ChEMBL.

### 3. A means of identifying which updated CIDX/SRC\_IDs did not change their iKeys.

Currently, for a given CIDX/SRC\_ID, if an updated CTAB is depostied, but the iKey generated from this CTAB is the same as the iKey for the previous version of the CTAB, then no changes are made in CR.

In order to identify such updated CTABs therefore, two fields exist in DEP\_COMPOUND\_CTAB \('IKEY\_CHANGED' and 'MD5\_CHANGED'\) which record whether the current CTAB or iKey is different to the previous \(now overwritten\) versions. A value of 'True' indicating that the item HAS changed, 'False' indicating that the item has NOT changed \(the previous and current versions of the item are the same\), and NULL indicating that the record has never been updated \(Only 1 version of this item has ever been deposited\). Note, therefore, that for a given record, it should never be the case that one of these fields is populated with 'True' or 'False' while the other is NULL.

With these data, we can specifically identify \(and exclude from the process\) CIDX/SRC\_IDs where NEITHER the iKey NOR the CTABs was changed after being updated \[ie: both '[\*]()\_CHANGED' fields are 'True'\]. This may occur if a depositor simply deposits exactly the same CTAB as before.

System Message: WARNING/2 \(scanForCleanerStructures.rst, line 126\); [_backlink_]()

 Inline emphasis start-string without end-string.

| MD5\_CHANGED | IKEY\_CHANGED | MEANING |
| :--- | :--- | :--- |
| null | null | A 'structure' for this CIDX/SRC\_ID has only ever been deposited once, and never updated. |
| True | True | An updated CTAB was deposited, and a different iKey was genearted from this. |
| True | False | An updated CTAB was deposited, but the same iKey was genearted from this. |
| False | True | Should never occur \[A different iKey has been obtained for an identical CTAB!\] |
| False | False | The 'updated' CTAB has not changed, and as expected, the iKey has not changed either. |
| True / False | null | Should never occur. |
| null | True / False | Should never occur. |

Note: If IKEY\_CHANGED is not null, then the IKEY\_DBVAL field should also be populated, with the iKey generated from the previous version of the CTAB. Obviously, if IKEY\_CHANGED is 'False', then IKEY\_DBVAL should not equal IKEY. The previous version of the MD5 is not captured.

