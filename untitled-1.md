# Depositing Activities against other Depositors Entities

Some depositors wish to deposit bioactivity data obtained using either assays \(AIDXs\) or compounds \(CIDXs\) defined by other depositors. Depositors should only do this if they are very confident that the other depositor has the capacity to maintain the identifiers correctly, and they are happy for the other depositor to update the definitions of these identifiers at any time without informing anyone else who may have deposited activity against these identifiers.

In order to use AIDXs and CIDXs owned by other depositors, these IDs must already have been deposited within ChEMBL by their respective owners.

Also, when citing these AIDXs and CIDXs in the Activity load file, the depositor must add an extra column to the file, giving the src\_id of the owner of these IDs.

