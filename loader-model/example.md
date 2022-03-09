---
description: >-
  An example load dataset, showing the structure of all required files and
  advice on avoiding common issues.
---

# Example Load Dataset

## Introduction to the load process

You can find the full specifications for the files [here](../untitled-10/). This page will cover the mandatory fields, foreign keys, and how the files relate to each other.

A simple set of example data files suitable for use as a template are available below:

{% file src="../.gitbook/assets/simple_full_example.zip" %}

### **REFERENCE.tsv**

```
RIDX	PUBMED_ID	JOURNAL_NAME	YEAR	VOLUME	ISSUE	FIRST_PAGE	LAST_PAGE	REF_TYPE	TITLE	AUTHORS	ABSTRACT
1								Dataset	Pathogen Box Compounds Screening 	Bloggs, J; Smith, M	400 compounds from the Pathogen box were screened for inhibitory activity against two human proteins...
```

The Reference file describes the deposition itself. Whether it currently has a published reference, or has been submitted pre-publication or without being associated with a publication.&#x20;

* **RIDX, YEAR** and **REF\_TYPE** are **mandatory**, all other fields are optional. So for example if you do not yet have a DOI, you can leave it blank. It can be added once your data are published.
  * For a depositied dataset, **YEAR** should be the year we recieved the file.&#x20;
* **REF\_TYPE** must be either "Publication" or "Deposited".

### ASSAY.tsv

```
AIDX	RIDX	ASSAY_DESCRIPTION	ASSAY_TYPE	ASSAY_TEST_TYPE	ASSAY_ORGANISM	ASSAY_STRAIN	ASSAY_TAX_ID	ASSAY_TISSUE	ASSAY_CELL_TYPE	ASSAY_SUBCELLULAR_FRACTION	TARGET_TYPE	TARGET_NAME	TARGET_ACCESSION	TARGET_ORGANISM	TARGET_TAX_ID
1	1	Compound was evaluated for the inhibition of human FECH at 10uM	B	In vitro	Homo sapiens		9606				PROTEIN	FECH	P22830	Homo sapiens	9606
2	1	Compound was evaluated for the inhibition of human HMBS at  100uM	B	In vitro	Homo sapiens		9606				PROTEIN	HMBS	P08397	Homo sapiens	9606
```

This file provides a brief description of the assay, along with the target organism, tissue, cellular fraction etc.

* **AIDX**, **ASSAY\_DESCRIPTION** _and_ **ASSAY\_TYPE** are all **mandatory**.&#x20;
* **RIDX** is optional, but if included it must be an RIDX owned by the depositor. Either one in this set of files or one already loaded into ChEMBL.
* This table describes what the assay is and what it targets.

#### **AIDX Definition**

* AIDX should not be duplicated  within a given source.&#x20;
* For example, if one panel of probes is being used on multiple sites, or be multiple collaborators, there should only be a single AIDX for "All experiments using this panel".&#x20;
* Alternatively, each site or group could have one AIDX that refers to "The experiment performed at this location".&#x20;
* Subsequent data that are using the same panel or at the same location should be loaded with the same AIDX.
* Ideally, an AIDX should be informative, to avoid mixups. e.g. a group leader's initials or a short version of the site name. It's a variable character field up to 200 characters.&#x20;

### ASSAY\_PARAM.tsv

```
AIDX	TYPE	RELATION	VALUE	UNITS	TEXT_VALUE	COMMENTS
1	CONC	=	10	uM		
2	CONC	=	100	uM		
```

This file describes the assay parameters. For example, here the first two records show the concentration of compound used.

* **AIDX** and **TYPE** are both **mandatory.**&#x20;
* It is a many-to-one mapping, so you can store multiple parameters for one assay.&#x20;
* Depositors can set their own **TYPE**, but the type must uniquely match to a data type. For example, you may not use both CONC and CONCENTRATION as TYPEs if they are both referring to the same sort of concentration data.&#x20;
* **AIDX** must match an existing AIDX owned by the depositor.

### COMPOUND\_RECORD.tsv

This is a truncated example showing only the first 3 rows. The full file is available in the "Simple Example Dataset" download.

```
CIDX	RIDX	COMPOUND_NAME	COMPOUND_KEY
MMV010764	1	MMV010764	MMV010764
MMV026468	1	MMV026468	MMV026468
MMV011229	1	MMV011229	MMV011229
```

* The **CIDX** field is **mandatory.**
* **RIDX** is optional, but if included it must be an RIDX owned by the depositor.&#x20;

### ACTIVITY.tsv

This is a truncated example showing only the first 3 rows. The full file is available in the "Simple Example Dataset" download.

```
RIDX	CRIDX	CIDX	AIDX	TYPE	RELATION	VALUE	UPPER_VALUE	UNITS	ACTIVITY_COMMENT
1	1	MMV161996	1	Inhibition		0		%	Not active
1	1	MMV202458	1	Inhibition		1		%	Not active
1	1	MMV676395	1	Inhibition		12		%	Not active

```

* **CIDX**, **AIDX** and **ACTIVITY** are **mandatory.**
* It is possible to load data without a CTAB file. If you include a CTAB file or will be loading structure data later, the CIDX fields in the CTAB **must** match the CIDX IDs here.
* **RIDX** is optional, but if included it must be an RIDX owned by the depositor.
* **ACT\_ID** is mandatory if providing an ACTIVITY_PROPERTIES or ACTIVITY_SUPPLEMENTARY record that maps to a given line.
* This table will take non-numerical activity values is submitted in the **ACTIVITY\_COMMENT** field.\
  **CRIDX** should generally be identical to **RIDX.**

### ACTIVITY\_PROPERTIES.tsv

```
ACT_ID	TYPE	RELATION	TEXT_VALUE	UNITS
1	HILL_SLOPE	=	1.1

```

_Not a mandatory file, and omitted from the simple test set._\
Can be used to supply additional activity information, for example the Hill slope of a curve. \
Should contain information that is neccessary for interpretation of the ACTIVITY table data. Raw results and other supporting information belongs in ACTIVITY\_SUPP.

* **ACT\_ID** and **TYPE** are mandatory.&#x20;
* **TYPE** descripes the type of measurement.&#x20;
  * Depositors can set their own categories for TYPE, but should not use the same TYPE value for different sorts of data.&#x20;
  * E.g. PB\_HILL\_SLOPE __ and PB\_R\_SQUARED would be valid, but storing both Hill slope and R-Squared data as PB would not be. Even if they had different UNITS or COMMENTS.
* Can contain dependent or independent variables. If RESULT\_FLAG is set to 1, it shows the record is a dependent variable/result (e.g., slope) rather than an independent variable/parameter.

### ACTIVITY\_SUPP

```
COMMENTS	REGID	RELATION	SAMID	TEXT_VALUE	TYPE	UNITS	VALUE
	0	=	0		Treatment		1
	1	=	1		Treatment		1
	2	=	2		Treatment		1
	2	=	2	Y	Side Effect (Y,N)	Y,N	
```

_Not a mandatory file, and omitted from the simple test set._\
This contains supplementary data on the **ACTIVITY** file.&#x20;

* It is not necessary to supply, for example, all the points on a curve.&#x20;
* It is more useful with datasets such as _in vivo_ studies where animal-level data is submitted.
* In this case, it shows the treatment group for each sample. And for any sample where a side effect on the cells was recorded, it notes this.

### ACTIVITY\_SUPP\_MAP.

```
ACT_ID	SAMID
0	0
1	1
2	2
```

_Not a mandatory file, and omitted from the simple test set._\
This table maps activity IDs in the **ACTIVITY** file to sample IDs in the **ACTIVITY\_SUPP** file. Both fields are **mandatory**.

### COMPOUND\_CTAB.sdf

This is a truncated example showing only the first SDF record. The full file is available in the "Simple Example Dataset" download.

* It is possible to load data without a CTAB file. If you include a CTAB file or will be loading structure data later, the CIDX fields in the CTAB **must** match the CIDX IDs in ACTIVITY.tsv.

```

  Mrv0541 04191616472D          

 27 28  0  0  0  0            999 V2000
   -5.0013   -1.2375    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -4.2868   -0.8250    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -4.2868    0.0000    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -3.5724   -1.2375    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -2.8579   -0.8250    0.0000 N   0  0  0  0  0  0  0  0  0  0  0  0
   -2.1434   -1.2375    0.0000 S   0  0  0  0  0  0  0  0  0  0  0  0
   -2.5559   -1.9520    0.0000 O   0  0  0  0  0  0  0  0  0  0  0  0
   -1.7309   -0.5230    0.0000 O   0  0  0  0  0  0  0  0  0  0  0  0
   -1.4289   -1.6500    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -0.7145   -1.2375    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    0.0000   -1.6500    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    0.0000   -2.4750    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    0.7145   -2.8875    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    1.4289   -2.4750    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    2.1434   -2.8875    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    2.1434   -3.7125    0.0000 O   0  0  0  0  0  0  0  0  0  0  0  0
    2.8579   -2.4750    0.0000 N   0  0  0  0  0  0  0  0  0  0  0  0
    3.5724   -2.8875    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    4.2868   -2.4750    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    5.0013   -2.8875    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    5.0013   -3.7125    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    5.7158   -4.1250    0.0000 Cl  0  0  0  0  0  0  0  0  0  0  0  0
    4.2868   -4.1250    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    3.5724   -3.7125    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
    2.8579   -4.1250    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -0.7145   -2.8875    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
   -1.4289   -2.4750    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  1  2  1  0  0  0  0
  2  3  1  0  0  0  0
  2  4  1  0  0  0  0
  4  5  1  0  0  0  0
  5  6  1  0  0  0  0
  6  7  2  0  0  0  0
  6  8  2  0  0  0  0
  6  9  1  0  0  0  0
  9 10  2  0  0  0  0
 10 11  1  0  0  0  0
 11 12  2  0  0  0  0
 12 13  1  0  0  0  0
 13 14  1  0  0  0  0
 14 15  1  0  0  0  0
 15 16  2  0  0  0  0
 15 17  1  0  0  0  0
 17 18  1  0  0  0  0
 18 19  2  0  0  0  0
 19 20  1  0  0  0  0
 20 21  2  0  0  0  0
 21 22  1  0  0  0  0
 21 23  1  0  0  0  0
 23 24  2  0  0  0  0
 18 24  1  0  0  0  0
 24 25  1  0  0  0  0
 12 26  1  0  0  0  0
 26 27  2  0  0  0  0
  9 27  1  0  0  0  0
M  END
> <CIDX>
MMV161996

$$$$


```

