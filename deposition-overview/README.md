# Basic Deposition and Overview

Loading data to ChEMBL is by invitation only. However, if you believe you have high quality bioactivity data that others would benefit from viewing in ChEMBL, then please contact us.

Below we provide a basic introduction to how BioActivity data should be formatted in order to be loaded into ChEMBL. Many aspects of this are best explained by reference to examples (see 'Examples' above).

## Depositors

All deposited data in ChEMBL must be associated with a source that has been registered in ChEMBL. Once such a source has been registered (and a src\_id \[a number] assigned), then depositors representing this source may submit data to the ChEMBL team who will load the data. The very first step to depositing data in ChEMBL, therefore, is to contact the ChEMBL team who will discuss your data with you and decide whether a new src\_id should be created for you to deposit your data against.

### Deposition Overview

You can see a flowchart explaining the deposition process below.&#x20;

{% embed url="https://docs.google.com/presentation/d/1b6AaDOhvSPmnaEbhYIoNHcUiTDqMHJNKwTq_NZPB9K8/edit?usp=sharing" %}

### Sources

All that is is required for a Source is a name and a short description.&#x20;

## Jobs

The loading of a collection of files for a single deposition is called a job and given a 'job\_id' if loading is successful.

* The job\_id associated with a DDI in ChEMBL is always the job\_id when the DDI _was first defined_.&#x20;
* Updating the definition of a DDI by including the DDI n a subsequent deposition job will _not change_ the job\_id associated with this DDI.

**Updating existing ChEMBL data using DDIs**

For example, a depositor uses a CIDX of, say, 'XYZ\_123' to define a particular chemical structure, but after loading to ChEMBL they subsequently discover that their structure is incorrect.

They can 'edit' the deposited structure by re-depositing XYZ\_123 with the new correct structure. A depositor can only edit data relating to their own identifiers. Thus, in the example above, if another depositor from a different src\_id owns a CIDX of 'XYZ\_123', then this will remain unaltered by the edits undertaken by the first depositor.

The field names of DDIs are always four letters, with the last three as 'IDX'. The 'X' is intended to indicate that the Identifier is an eXternal identifier, not an identifier internal to ChEMBL.

