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
| Field Definitions and input constraints for input files.  | input\_files | 193 |
| Field Existence constraints to use. | input\_files | 193 |
| Field Dependency constraints to use, where field A is dependent on field B. | \(d1, d2, d3...\), input\_files | 96, 193 |
| Other field constraints \(e.g. where 2 columns may not both be NULL\) | fieldConstraintsDict | 338 |

**globalVars.py**

| Definition | Variable Name | Approx. Line |
| :--- | :--- | :--- |
| Field Dependency rules | gd1, gd2... | 2788 |
| Mappings of DEP\_ table fields to ChEMBL table fields | map2C | 583 |
| Target fields to used to get a target ID | self.TARGET\_... | 199 |
| Filename patterns to use for input files | self.INFO etc. | 78, at comment "Deposition Files..." |
| Patterns defining whether the content of a field is valid. | gp001, gp002.... | 2969 |
| TRVU header label definitiions | getTRVUdics | 1454 |
| Table headers added during analysis steps | anaD | 319 |
| ChEMBL tables used by the  loader scripts |  | 184 |
| ACTIVITY header definitions | self.TYPE... | 62 |

**settings.py**

| Definition | Variable Name | Approx. Line |
| :--- | :--- | :--- |
| List of DBs available for 'internal' administrators. | PERMITTED\_DB_\__LIST\_TUP | 60 |
| Permitted host addresses for the webapp | ALLOWED\_HOSTS | 93 |
| DB engine, name, user and password | DATABASES | 154 |

There are lots more settings in the Django settings file, but these are the ones you are most likely to need to change when moving to a new DB or host.

