# Depositor-Defined Identifiers

Identifiers for key entities such as Compounds and Assays are defined within the deposition job files by using 'Depositor-Defined Identifiers' \(DDIs\). These identifiers are created and maintained by depositors themselves, and provide a way for depositors to:

* Give a meaningful unique identifier to for each compound and assay in their datasets.
* Update sets later with additional data.

Depositor-defined Identifiers are used for Compounds/Substances, Assays and References, and are called CIDX, AIDX and RIDX, respectively.

* Depositors may use any UniCode characters to define these identifiers.
* Identifiers must consist of a string of between 1 and 200 visible characters.
* DDIs are 'owned' by the depositor. In other words, a given identifier is defined by the depositor when a deposition is first made with the identifier.
* Subsequent use of the same identifier in future ChEMBL loads will result in the updating of the data that was originally associated with this identifier when it was first loaded.



### Sources with multiple depositors

* It is possible for a source to have multiple depositors, for example a large study that tests the same panel of drugs at several labs on several sites. 
* The depositors can share a RIDXs and CIDXs, but the AIDX must be unique for each assay deposited.
* To avoid duplication within a source, we recommend that each site, research group etc adds a prefix to the AIDX. The AIDX field is a 200-character field, so this can easily be a name. 

