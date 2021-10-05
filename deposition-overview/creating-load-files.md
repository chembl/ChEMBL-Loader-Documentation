# Creating Load Files

### Basics

Currently, ChEMBL only accepts data formatted into tsv \(tab-separated variable\) files. The files must be named as described in the 'Requirements' section. Within these files, column headers such as 'CIDX' and 'AIDX' are used to indicate the column position of the Depositor defined IDs. The 'Requirements' link also describes what other fields are required for these files. Some of these fields are mandatory, others optional.

Some files are termed 'Primary' deposition files. These files are responsible for defining the key DDIs \(CIDX, AIDX, and RIDX \[see below\]\). 'Secondary' files add extra information about DDIs defined in corresponding Primary files. Thus the ASSAY\_PARAM file can be used to assign a list of parameters to an AIDX \(which must have either been previouslhy deposited in ChEMBL, or be defined in the ASSAY file \[or both\]\).

As can be seen from the 'Requirements' link, very many combinations of files can be used in a single deposition, provided that the referential integrity of DDIs is maintained.

The collection of files used in a single deposition is assigned a 'job\_id'. Each deposition job either fails completely or succeeds completely. In other words, either **all** the data from a job is loaded, or **none**.

Using the 'Gateway' link below allows depositors to test if their files are suitability formatted for loading to ChEMBL.

### Some possible Complexities

#### Reference IDs

A third Depositor-Defined Entity is called RIDX. RIDXs are used to define a Reference of some kind. This may be a publication in the literature, or a resource on the internet. So, if required, depositors may create multiple RIDXs and associate them with different CIDXs, AIDXs and Activities.

However, the RIDX is not mandatory, and so for most simple depositions it can ignored. Indeed for many depositors, there is no requirement to associate references with data.

Note that all src\_ids, when first created by the ChEMBL administrator, are assigned a 'default RIDX'. If no RIDX is defined in a job, then this 'default RIDX' is used instead.

#### [Complex Result Sets](../design-specification-and-internal-docs/complex-results.md)

ChEMBL has the ability to store more complex result sets than has hitherto been possible. Supporting Raw data, Multiple Result Types, Properties, etc can all be captured.

#### [Depositing Activity Data against other depositors' entities](../untitled-1.md)

Some depositors wish to deposit bioactivity data obtained using either assays \(AIDXs\) or compounds \(CIDXs\) defined by other depositors. In order to do this, these AIDX and CIDXs must already have been deposited within ChEMBL by their respective owners, and when citing these AIDXs and CIDXs in the Activity load file, the depositor must add an extra column to the file, giving the src\_id of the owner of these IDs.

## Testing your files.

Visit the [Gateway](https://wwwdev.ebi.ac.uk/chembl/gateway/gateway/) website, choose the files you wish to test, and then hit 'upload and process'

## Submitting your files for loading.

Once you are confident that your files pass the formatting tests above, create a zip or tar archive of your files, and email it to us. If you are a registered depositor then we will load the file, and send you back a load report. If you are not a registered depositor, then we will consider whether your data is appropriate for ChEMBL, and be in contact with you shortly.

