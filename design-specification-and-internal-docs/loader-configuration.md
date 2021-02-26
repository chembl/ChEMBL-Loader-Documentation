---
description: >-
  Some of the settings may need changing as the ChEMBL schema evolves or as it
  is forked for use on other projects. This page describes where the main
  configuration options can be found
---

# Loader configuration

**ldConfigs.py**

| Definition | Variable Name | Approx. Line |
| :--- | :--- | :--- |
| File existence constraints and presentation order for results. | files | 122 |
| Self-Referenced Files | SRkeys | 572 |
| Primary files \(must be present in a load\) | files1 | 528 |
| Secondary, Tertiary and Quaternary files \(may be present in a load\) | files2, files4 \(files3 does not exist as the only tertiary file is ACTIVITY\) | 540, 550 |
| Field Definitions and inpur constrains for input files.  | input\_files | 193 |
| Field Existence constraints to use. | input\_files | 193 |
| Field Dependency constraints to use, where field A is dependent on field B. | \(d1, d2, d3...\), input\_files | 96, 193 |
| Other field constraints \(e.g. where 2 columns may not both be NULL\) | fieldConstraintsDict | 338 |

**globalVars.py**

| Definition | Variable Name | Approx. Line |
| :--- | :--- | :--- |
| Field Dependency rules | gd1, gd2... | 2788 |
| Mappings between different tables that store the same data, |  |  |
| Target fields to use from input files |  |  |
| Filename patterns to use for input files | self.INFO etc. | 78, at comment "Deposition Files..." |
| Patterns defining whether the content of a field is valid. | gp001, gp002.... | 2969 |
| TRVU header label definitiions | getTRVUdics | 1454 |
| Table headers added during analysis steps | anaD | 319 |

