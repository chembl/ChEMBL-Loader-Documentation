# newChemblrequirements.rst

Following a recent review of ChEMBL, a series of requirements were identified for how ChEMBL might be updated and enhanced. The following is a summary list of the main requirements that have been identified..

1. A Common loading framework for all bioactivity data \(ie: Literature and Non-Literature BioActivity data\).
2. Updating functionality. All loaded data should be easily ‘updatable’ by the depositor.
3. Flexible Formatting of deposition data sets. Loader must permit some flexibility in formatting. Eg: xls, tsv or pipe-separated \(and ability to add others in future \[eg: JSON etc\] .. but NOT MySQL insert statements!!! \), column order irrelevant, optional fields, etc.
4. Flexible entity deposition ie: The ability to load just compounds, or just assays, or just references, or just activities, or any combination of these in a single deposition.
5. Load data against entities defined by other depositors. For example, a depositor X should be able to load a set of Activities for compounds defined either by depositor X, Y or Z, tested in assays defined either by depositor X, Y or Z.
6. Efficient Error Reporting. Optimize parsing to maximize the capture of as many warnings and errors as possible, to minimize the iterations required for a user to ‘correct’ their data. Include a reporting of ranked errors and warnings.
7. All deposited \(ie: Pre-curated\) data captured and stored unchanged.
8. All pre-curated deposited data should be available to be displayed in the user interface, if required.
9. Monitoring of Updates. Web pages to display reports of recent loads and processes. Accessible to all ChEMBL internally.
10. Deposition Web Portal for all Load Types - Available external to EBI. To allow users to test their deposition sets for correct formatting / integrity etc \[– but not to allow them to load it themselves ! \]. Including Documentation on preparing Deposition Sets, and example data sets.
11. Fast release cycle. Aim to have ChEMBL in a ‘Release-ready’ state as the default state \(... or at least weekly ?\).
12. Embargo utility. An ability to ‘embargo’ a data set \(Only authenticated users to view, in the ChEMBL web interface, embargoed data from a particular source\).
13. Tagging data sets. The ability to ‘tag’ or ‘label’ a set of deposited data within ChEMBL for the convenience of a depositor to reference. Tagged data sets \(the precurated, non-normalized set\) should be 'exportable', and stable identifiers should exist for them \(CHEMBL\_IDs\).
14. Frozen Data Sets. The ability to ‘freeze’ or ‘lock’ a tagged data set \(ie: prevent it from being updated further\).
15. The ability to handle Complex Result Sets \(ie: The ability to... store all raw data, handle multiple result types for a single assay and permit result types to have 'properties' to qualify or contextualize the result value\).

