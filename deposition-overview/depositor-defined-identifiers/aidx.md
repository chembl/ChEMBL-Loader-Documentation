---
description: Depositor-defined Assay IDs
---

# AIDX

### AIDX

* Depositor-defined ASSAY IDs (AIDXs) are defined in ASSAY deposition files.
* Each line in an ASSAY file records information on the protocol used for the experiment.&#x20;
* An AIDX should be a string that is meaningful to the depositor, and must be unique within the given source.
* Each assay can only have a single target.
* In the case of a panel assay, there will need to be one Assay record per target.
* Standard reference assays that are used repeatedly can be deposited as a specific dataset, with just that one assay/group/panel.

### **Tracking similar assays in ChEMBL using the AIDX**

The AIDX is a depositor identifier that is provided alongside depositor assays. It should be concise and meaningful and not easily replicated by different depositors.\
Each AIDX corresponds to a distinct assay set-up with the aim, target and method defined in the assay description.

#### Suggested format:&#x20;

Smith\_KinaseXYZ\_Assay36\
(abbreviation of the group leader, target, assay number). This AIDX is easy to compare with others, and gives an intuitive idea of what sort of assay it is. It makes any data issues during loading easier to resolve, and makes your data easier to search when loaded.

#### Poor format:&#x20;

1’, ‘2’, ‘3’

These AIDXs do not give any information about the assay.&#x20;

### When can additional activity data be submitted to an existing assay?

In rare cases, depositors submit additional bioactivity data for existing assays.For example, when a single institute is performing the same set of assays for newly synthesised compounds, or a contract research organisation is characterising compounds.&#x20;

An existing AIDX can be recorded alongside the new bioactivity data and all ASSAY fields for the record (ASSAY\_DESCRIPTION etc) must match the previous AIDX exactly.  Depositors are responsible for ensuring that the assay set up is consistent with a standard protocol, and ideally with appropriate positive and negative controls to ensure variation is within expected limits.

### How do I record an assay that uses a commercial kit?

Commercial kits have a standard assay set-up but we consider assays run by different institutions, using the same kit, as distinct. To allow comparison of similar assays, kit identifiers/catalogue numbers should be included in the ASSAY\_DESCRIPTION field since there are sometimes country specific kits or testing locations.

e.g. Inhibition of XYZ target at 10uM tested using the Eurofins SafetyScreen44 (BI) (P270)  P270 is the catalogue number.

\
\
