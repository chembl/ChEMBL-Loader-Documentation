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
| Primary files \(must be present in a load\) | files1 |  |
| Field Definitions for input files.  | input\_files | 193 |
| Field Existence constraints to use. |  |  |
| Field Dependency constraints to use, where field A is dependent on field B. | \(d1, d2, d3...\), input\_files | 96, 193 |
| Other field constraints \(e.g. where 2 columns may not both be NULL\) |  |  |

**globalVars.py**

| Definition | Variable Name | Approx. Line |
| :--- | :--- | :--- |
| Dependency rules \(used in LDConfigs\) |  |  |
| Functions called by Field Dependency constraints |  |  |
| Mappings between different tables that store the same data, |  |  |
| Target fields to use from input files |  |  |
| Filename patterns to use for input files | self.INFO etc. | 78, at comment "Deposition Files..." |

