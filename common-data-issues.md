# Common data issues

## Placeholder values

You don't generally need to use these. Very few fields are mandatory, and so you can leave a field empty if necessary.

## Illegal XIDX values

Some values are not permitted for AIDX/RIDX/CIDX. Currently these are

* CR0
* default

## Unpublished data

You can send us it as a dataset and we'll assign a DOI If it's accepted, then send us the journal details and we'll assign a DOI.&#x20;

If you do have a DOI but it's not actually published, then send us all the details.&#x20;

If you need a reviewer to be able to see the data during review, then contact us and we can discuss hosting.

## Incomplete datasets

**The COMPOUND\_CTAB  file contains CIDXs that are not in COMPOUND\_RECORD/Not in ACTIVITY**

* If this results in an ACTIVITY with no legal COMPOUND\_RECORD entry, it'll cause a CIDX/CRIDX error.
* If there is a CIDX in the CTAB that is not re\[resented in in the COMPOUND\_RECORD, we get _"CIDX is a self-referencing ID in this file, so must exist in accompanying corresponding primary file or DB"_
* Under normal operation, the loader will issue a fatal error (penalty score = 9) if a CIDX in the COMPOUND\_CTAB does not exist in either the accompanying COMPOUND\_RECORD file or in the database.

**The COMPOUND\_CTAB  file does not contain all the structures in COMPOUND\_RECORDS**

* This will load correctly, but the compounds will be missing structures unless a later load using this dataset is done using the CTAB and whatever minimal data are needed to get it to load (I beleive this may be just the reference, I need to test this)
* This does not throw a warning. A warning will be added to the loader.
