# Depositor-Defined Identifiers

Identifiers for key entities such as Compounds and Assays are defined within the deposition job files by using 'Depositor-Defined Identifiers' (DDIs). These identifiers are created and maintained by depositors themselves, and provide a way for depositors to:

* Give a meaningful unique identifier to each compound and assay in their datasets.
* Update sets later with additional data.

Depositor-defined Identifiers are used for Compounds/Substances, Assays and References, and are called CIDX, AIDX and RIDX, respectively.

They are also used for internal reference betoween records in a dataset. REGID and SAMID are depositor-defined identifiers used to group together supplementary data. SAMID groups supplementary rows that refer to the same activity row

* Depositors may use any UniCode characters to define these identifiers.
* Identifiers must consist of a string of between 1 and 200 visible characters.
* DDIs are 'owned' by the depositor. In other words, a given identifier is defined by the depositor when a deposition is first made with the identifier.
* Subsequent use of the same identifier in future ChEMBL loads will result in the updating of the data that was originally associated with this identifier when it was first loaded.

Other Depositor-defined IDs include SAMID, used to link ACTIVITY\_SUPP records to ACTIVITY rows, and REGID, used to group ACTIVITY\_SUPP data.

### Sources with multiple depositors

* It is possible for a source to have multiple depositors, for example a large study that tests the same panel of drugs at several labs on several sites.&#x20;
* The clearest example of this is Source 52, which contains SARS-CoV-2 assay data from multiple depositors.
* The depositors can share RIDXs and CIDXs, but the AIDX must be unique for each assay deposited.
* To avoid duplication within a source, we recommend that each site, research group etc adds a prefix to the AIDX. The AIDX field is a 200-character field, so this can easily be a name.&#x20;

## Depositor-Defined IDs



ChEMBL uses unique identifiers, defined by a depositor, to identify Assays, Compounds and References within each given Source.&#x20;

The names (or 'headers' in a deposition file) for such identifiers are of the form ‘\*IDX’ where the ‘\*’ may be any capital letter, and ‘X’ refers to the fact that it is externally managed. This format is designed to clearly distinguish this class of identifiers from other (internal) ChEMBL identifiers.

### Types of Depositor-Defined ID

Currently there are three such \*IDX’s, covering the 3 main entity types in ChEMBL: Assays, Compounds and References. Each of these are defined in a 'primary' deposition file, as show below...

| [**\***](broken-reference)**IDX** | **Description**                | **Primary File** |
| --------------------------------- | ------------------------------ | ---------------- |
| **AIDX**                          | Depositor-defined Assay ID     | ASSAY            |
| **CIDX**                          | Depositor-defined Compound ID  | COMPOUND\_RECORD |
| **RIDX**                          | Depositor-defined Reference ID | REFERENCE        |

Depositors may use up to 200 characters to define \*IDX’s. Depositors are free to use whatever UniCode characters they wish to use (but non-visible characters will be stripped from both ends). All deposited files must be ascii or UTF-8 encoded.

###
