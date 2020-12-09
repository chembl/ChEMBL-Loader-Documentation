# Introduction to the ChEMBL Loader Gateway

The ChEMBL Gateway aims to satisfy a number of requirements for loading and storing data in ChEMBL

Although loading data to ChEMBL will remain by invitation only, and will remain in the complete control of ChEMBL administrators, the external facing 'Gateway' aims to stimulate external requests, and assist depositors in correctly formatting depositions before submission to ChEMBL.

By way of introduction, Below are listed some of the basic key concepts used in the new ChEMBL loader. Other documentation pages go into many of these concepts in more detail, and some complex aspects are dealt with by examples \(see 'Examples' above\).

## JOBS

The loading of a collection of files for a single deposition is called a job and given a 'job\_id' if loading is successful.

The job\_id associated with a Depositor defined ID in ChEMBL is always the job\_id when the Depositor defined ID _was first defined_. Thus, updating the definition of a Depositor defined ID by including the Depositor defined ID in a subsequent deposition job will _not change_ the job\_id associated with this Depositor defined ID.

## Depositor-Defined Identifiers: CIDX, AIDX and RIDX

Depositor-defined Identifiers are used for Compounds/Substances, Assays and References, and are called CIDX, AIDX and RIDX, respectively.

* Depositors may use any UniCode characters to define these identifiers.
* The most important aspect of these identifiers that depositors must understand is that they are 'owned' by the depositor. In other words, a given identifier is defined by the depositor when a deposition is first made with the identifier.
* Subsequent use of the same identifier in future ChEMBL loads will result in the updating of the data that was originally associated with this identfier when it was first loaded.

## Load files

Load files must be ascii or UTF-8 encoded, and named and formatted as described in the 'Requirements' link above. Within these files, column headers such as 'CIDX', 'AIDX' and 'RIDX' are used to indicate the column position of the Depositor defined IDs. Depositor defined IDs may be consist of a string of upto 200 UniCode characters

* For each of these three key entities a single \('primary'\) file is used to define \(or redefine\) them, as shown in the 'Requirements' link above.
* Other \(secondary\) files may define or redefine properties of existing Depositor defined IDs \(ie: which either exist in the corresponding primary file in the same deposition job, or already exist in the database\).
* Activity files must also cite only existing Depositor defined IDs.

Currently, ChEMBL only accepts data formatted into tsv \(tab-separated variable\) files, but alternative formats will be available in the future..

## [Complex Result Sets](design-specification-and-internal-docs/complex-results.md)

Activity data that is more complicated than a simple 'cpdX has affinity Y nM against target Z' can now be accommodated in ChEMBL.

## [Compound Normalization](design-specification-and-internal-docs/untitled-12.md)

A Compound Normalization procedure is run separately from loading processes, and now includes an assassment of the quality of the drawn structures.

## Depositors

A depositor is equivalent to a src\_id. In practice, multiple individuals within and organization may be recognized as individuals permitted to deposit data on behalf of a src\_id. The management of multiple depositors for a single src\_id is not dealt with by ChEMBL, but is the responsibility of the ChEMBL administrators.

Every src\_id is also assigned a 'default RIDX' which is used as the RIDX if the load files do not specify an alternative RIDX.

## [Embargoing](design-specification-and-internal-docs/embargo-tagging.md)

Embargoing is managed by administrators by appending a '-E date' option when loading a job.

## [Tagging and Freezing](design-specification-and-internal-docs/embargo-tagging.md)

Tagging and Freezing of 'sets' of deposition job\_id is achieved with the loader, but the process is dealt with separately from the loading process itself.

