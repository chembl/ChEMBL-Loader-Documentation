# Basic Deposition and Overview

## Basic Deposition and Overview

Loading data to ChEMBL is by invitation only. However, if you believe you have high quality bioactivity data that others would benefit from viewing in ChEMBL, then please contact us.

Below we provide a basic introduction to how BioActivity data should be formatted in order to be loaded into ChEMBL. Many aspects of this are best explained by reference to examples \(see 'Examples' above\).

## Depositors

All deposited data in ChEMBL must be associated with a source that has been registered in ChEMBL. Once such a source has been registered \(and a src\_id \[a number\] assigned\), then depositors representing this source may submit data the the ChEMBL team, who will load the data. The very first step to depositong data in ChEMBL, therefore, is to contact the ChEMBL team who will discuss your data with you and decide whether the a new src\_id should be created for you to deposit your data against.

## Depositor-Defined Identifiers

Identifiers for key entities such as Compounds and Assays are defined within the deposition job files by using 'Depositor-Defined Identifiers', called CIDX and AIDX respectvely. Depositor-Defined Identifiers must consist of a string of between 1 and 200 visible characters \[UTF-8 encoding required\].

As the term implies, these identifiers are created and maintained by depositors themselves, and provide a straightforward mechanism for depositors to update or correct previously deposited data. In other words, if a depositor uses a CIDX of, say, 'XYZ\_123' to define a particular chemical structure, but after loading to ChEMBL they subsequently discover that their structure is incorrect, then they can 'edit' the deposited structure simply by re-depositing XYZ\_123 with the new correct structure. Clearly, a depositor can only edit data relating to their own identifiers: Thus, in the example above, if another depositor from a different src\_id owns a CIDX of 'XYZ\_123', then this will remain unaltered by the edits undertaken by the first depositor.

The names of Depositor-Defined Identifiers are always four letters, with the last three as 'IDX'. The 'X' is intended to indicate that the Identifier is an eXternal identifier, not an identifier internal to ChEMBL.

## Creating Load files.

### Basics

Currently, ChEMBL only accepts data formatted into tsv \(tab-separated variable\) files. The files must be named as described in the 'Requirements' link above. Within these files, column headers such as 'CIDX' and 'AIDX' are used to indicate the column position of the Depositor defined IDs. The 'Requirements' link also describes what other fields are required for these files. Some of these fields are mandatory, others optional.

Some files are termed 'Primary' deposition files. These files are responsible for defining the key Depositor-Defined Identifiers \(CIDX, AIDX, and RIDX \[see below\]\). 'Secondary' files add extra information about Depositor-Defined Identifiers defined in corresponding Primary files. Thus the ASSAY\_PARAM file can be used to assign a list of parameters to an AIDX \(which must have either been previouslhy deposited in ChEMBL, or be defined in the ASSAY file \[or both\]\).

As can be seen from the 'Requirements' link, very many combinations of files can be used in a single deposition, provided that the referential integrity of Depositor-Defined IDs is maintained.

The collection of files used in a single deposition is assigned a 'job\_id'. Each deposition job either fails completely or succeeds completely. In other words, either **all** the data from a job is loaded, or **none**.

Using the 'Gateway' link above allows depositors to test if their files are suitability formatted for loading to ChEMBL.

### Some possible Complexities

#### Reference IDs

A third Depositor-Defined Entity is called RIDX. RIDXs are used to define a Reference of some kind. This may be a publication in the literature, or a resource on the internet. So, if required, depositors may create multiple RIDXs and associate them with different CIDXs, AIDXs and Activities.

However, the RIDX is not mandatory, and so for most simple depositions it can ignored. Indeed for many depositors, there is no requirement to associate references with data.

Note that all src\_ids, when first created by the ChEMBL administrator, are assigned a 'default RIDX'. If no RIDX is defined in a job, then this 'default RIDX' is used instead.

#### [Complex Result Sets](complex-results.md)

ChEMBL has the ability to store more complex result sets than has hitherto been possible. Supporting Raw data, Multiple Result Types, Properties, etc can all be captured.

#### [Depositing Activity Data against other depositors' entities](untitled-1.md)

Some depositors wish to deposit bioactivity data obtained using either assays \(AIDXs\) or compounds \(CIDXs\) defined by other depositors. In order to do this, these AIDX and CIDXs must already have been deposited within ChEMBL by their respective owners, and when citing these AIDXs and CIDXs in the Activity load file, the depositor must add an extra column to the file, giving the src\_id of the owner of these IDs.

## Testing your files.

Visit the Gateway link above, choose the files you wish to test, and then hit 'upload and process'

## Submitting the files for loading.

Once you are confident your files pass the formatting tests above, then create a tarball of your files, and email it to us. If you are a registered depositor then we will load the file, and send you back a load report. If you are not a registered depositor, then we will consider whether your data is appropriate for ChEMBL, and be in contact with you shortly.

