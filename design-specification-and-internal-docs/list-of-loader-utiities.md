---
description: List of loader utilities for cloader.py
---

# List of Loader Utilities

Utils are marked as (tested) if James has run them to check they generate apparently valid output.

Utils are marked as \[CURRENTLY UNAVAILABLE] if they are broken or not yet implemented.

Utils are marked as \[NOT FULLY TESTED] if Jon did not finish testing them before he left in 2018.

## 1 to 20 : Process deposition data&#x20;

### 1: Load

Process deposition data. Requires -m, -L \[-d]

### 2: Correct \[CURRENTLY UNAVAILABLE] \[Future Work]

Make a correction to some deposited data. Only 1 file at a time .. update ONLY fields shown.Ie: just TITLE to correct a spelling error in the title ? IGNORE all null or empty fields.

### 10: Compound Normalization:

Normalize recently loaded compounds. The normal operation for the 'Compound Normalization' utility in the loader is to:

* First run a DB query to identify all CIDX/SRCID combinations requiring normalization.&#x20;
* Then, process these in batches of 10 through a normalization process and commit at the end of each batch.&#x20;
  * The commit step requires a response from the user, although if the '-a' option is used then the default is used instead ('yes, commit' in this case).&#x20;
  * Processing of batches will continue until all CIDX/SRC\_IDs have been normalized.

**Options**

* If the user wishes to change the batch size from 10, then they can do this with the '-y' (lower case) option (which takes an integer).&#x20;
* If the user wishes to limit the total number of CIDX/SRC\_IDs processed in one run of the script, then this may be altered in a similar way by using the '-Y' option (uppercase).&#x20;

Thus, for example, running with the options ' -y 20 -Y 502 ' will result in the processing of 502 CIDX/SRC\_IDs, 20 at a time (ie: 26 batches, with 20 CIDX/SRC\_IDs in all but the last batch, which would contain 2 CIDX/SRC\_IDs).

**Webservice requirements**

* The standardization process included in the normalization process requires the calling of the ChEMBL RDKit web-service.&#x20;
* If a '500' error is returned during any single batch then the normalization r un is aborted.&#x20;
* Setting -Y and -y to reasonably low numbers (and perhaps stringing together multiple script runs consecutively on the LSF) mitigates the risk of network interruptions resulting in the 'loss' of large numbers of normalizations before they can be committed.&#x20;
*   However, the cost of setting -Y and -y too low, is that:&#x20;

    * 1\. Each run of the Normalization process requires a full query o f the CR table to assess which CIDX/SRC\_IDs currently require normalization**.**
    * 2\. Each batch req uires a single run of the InChI generation software (it is far more efficient to run this for multiple structures simultaneously.)



### 14: Scan for Cleaner Structures (tested)

**NOTE: This runs, but we need to run a test that ut deifnitely corrects structures. E.g. run against known-bad data with known-good deposited structures from another source.**\
Run a search for deposited structures which have lower penalty scores (from the 'structure checker' web service) than the current ChEMBL CTAB for the same InChIKey

* Requires -J

_Example: _python manage/cloader.py -d 137 -u 14 -J 29 -v 5 -a

### 17: Delete Activities

Delete all 'Activities' data relating to a given LIST of EITHER

1. AIDX's from file '-f'
2. 2.JOB\_IDs on command line ('-J'). '-s'

The owner of the AIDX's or JOB\_ID's, must also be specified.&#x20;

* Note that ALL related SUPPLEMENTARY and PROPERTY data is also deleted, as is all corresponding data in 'DEP' tables.&#x20;
* Tables affected: ACTIVITIES, ACTIVITY\_PROPERTIES, ACTIVITY\_SUPP, ACTIVITY\_SUPP\_MAP, ACTIVITY\_SMID, DEP\_ACTIVITY, DEP\_ACTIVITY\_PROPERTIES, DEP\_ACTIVITY\_SUPP, DEP\_ACTIVITY\_SUPP\_MAP&#x20;
* Note also that certain ChEMBL tables are connected to 'parent' tables with a 'cascade delete' in place, so are automatically deleted if parent PKs are deleted. Thus, 'ACTIVITY\_SUPP\_MAP' and 'ACTIVITY\_RGID\_MAP' are cascade connected to 'ACTIVITY\_SMID', and 'ACTIVITY\_PROPERTIES' to 'ACTIVITY'.

## 21 to 40 : Job management

### 21: Show jobs (tested)

Show all jobs. Filter to show only for one source if '-s' option used. Requires -d. add -v for all fields. Use -J to limit to a range of JOBIDs

_Example: _python manage/cloader.py -d 137 -u 21 -v 5 -a

### 22: Report job(s) in sets (tested)

Show all the jobs, and the sets they belong to, for either a single job (as -J int), a list of jobs (as -J in a hyphen separated list), or a single set (as -Set int). Requires -J or -Set. Ifquerying with a single set, then CHEMBLID may be used instead of set\_id. Note that in the resultsshown, a single job will be listed more than once if it belongs to multiple sets.

_Example:_ python manage/cloader.py -d 137 -u 22 -v 5 -a -J 29

### 23: Export job(s) (tested)

Export all data for either a single job (as -J int), a list of jobs (as -J in a hyphen separated list) or a single set (as -Set int). If single set, then CHEMBLID may be used instead of set\_id. Requires -J or -Set, and -O \[a valid directory, or, to print to screen, use one of either '-','none' or 'screen']. If separate files required for separate jobs, then use the -M option, to create subdirs named 'jobID\d+' in -O

_Example:_ python manage/cloader.py -d 137 -u 23 -v 5 -a -J 29 -O /nfs/panda/chembl/chembl\_loader/cloader\_example\_output/util23

### 24: Show job load report (tested)

Show the report on a load for a single JOB\_ID (-J). Requires -d

_Example: _python manage/cloader.py -d 137 -u 24 -v 5 -a -J 29

### 25: Show content report (Jobs or AIDXs) (Tested)

Show a report of the content assigned to JOB\_ID(s) (-J, eg: -J '1-4,6,8-9'), or a list of AIDXs present in a text file, location '-f'. Requires -d, -s

_Example: _python manage/cloader.py -d 137 -u 25 -v 5 -a -J 29 -s 95

## 121 to 140 : Sources&#x20;

### 121 Create a new Source:

Create a new SRC\_ID. Requires -d, -s (new src\_id), -n (new name), -Z (new description),-y (year)

Will NOT work on CHEMLDT as that database has no CHEMBL\_ID\_LOOKUP table

_Example: _python manage/cloader.py -d 137 -u 121 -s 99 -n TEST -Z TESTING -y 2021

## 141 to 160 : Processing, Analysis, etc.&#x20;

### 141: Get Activity Data \[NOT FULLY TESTED] (BUGGED, something to do with the args)

Print Activity data for a list of Activity\_ids or a Job\_ids (Requires -d, -p and -J or -A).

* '-p' is a pipe separated property list with 3 elements (0-2), each used to indicate what should be included (eg: '-p APM|IN|T')
  * 0 = 'Which Tables to Query'
    * A = ACTIVITIES&#x20;
    * P = ACTIVITY\_PROPERTIES (Shown separately with 'T' but merged onto A with 'M')
    * M = ACTIVITY\_SUPP\_MAP (Only shown separately for 'T' option).
    * E = ACTIVITY\_SMID (Only shown separately for 'T' option)
    * S = ACTIVITY\_SUPP, Excluding records with null SMIDs
    * U = ACTIVITY\_SUPP, but Include records with nUll SMIDs. Only affects 'M'.
    * Note: No P without A, and no U without S.
  * 1 = 'Which Columns and Cells to Show'
    * \[NB: These do not apply to 'T' option below]
    * I = Identifiers (Include SMIDs).
    * r = show raw (pre-normalized) data.
    * R = show raw (pre-normalized) data, and pivot on these data.
    * n = show normalized form of data.
    * N = show normalized form of data, and pivot on these data.
    * Z = Show Full SMID data from supp in ACT matrix \[otherwise just show SMID values if S selected above].
    * C = Compact. If the values in a 'units' or 'relation' column are all the same, then put these into the 'type' header, and omit the column from the output.
    * F = Fill pivot gaps. Fill the gaps you'd normally have in pivot table with appropriate data.
    * t = transpose table (by 90 degrees : headers become row names)
  * 2 = 'Output Format'.\[NB: choose either M or T]
    * T = Tables, as per database, nothing added or subtracted and no format changes.
    * M = Matrix (2D). Make 3 matrices: 'ACT', 'PROP' and SUPP' 2D matrices. If no numbers, then make ALL, else make only numbered ones.
      * 1\. 'ACT' matrix is a P and S appended as extra cols onto A.
      * 2\. 'PROP' matrix is a pivot of P onto A with Types as headers, using ACTIVITY\_IDs as row numbers.
      * 3\. 'SUPP' matrix is a pivot of S/U alone with Types as headers, using RGID as row numbers. Will be created from S or U, whichever is selected.
  * Data is printed to console by default.&#x20;
    * Use '-y' \[int] to constrain col spacing, '99' to autofit.
    * Use '-Y' \[int] to constrain col1 spacing (best for transformed data only).
    * Use '-Z' \[str] show only these cols. eg: -Z '2-5|8|11|14|17-20|23-28|34'.
    * First col is '0'. Show all cols if -Z not used.
    * Use '-X' \[boo] to truncate printing to console so lines do not overrun to the next line.
    * Use '-Q' \[str] for an alternative to '|' as a separator for 'Identifiers' (see above) if multiple exist.
    * Use '-O' \[str] for printing to file instead of console \[give directory ('.' if current dir)].
    * Use '-z' \[int] to limit all queries using 'rownum < z' \[useful to see T structure]
  * Comparisons to original data may be required
    * Use '-k' \[str] path to 'kLol' (a picKled 'List of lists' \[usually captured spreadsheet]) data. Show as a separate matrix after matrix 3.
    * Use '-K' \[boo] merge M3U (Matrix 3 using 'U' query) INTO a copy of kLol(from '-k' above), and show as another separate matrix (requires '-k').
    * Note : it is inadvisable to select AS and Z unless 'G' is true \[ie: Full Granular Mapping curation has been carried out].Otherwise, if very many A records are each mapped many or all S records, then theACT matrix could be very large indeed, as there will be a separate row for each A -to- S combination.



## 901 to 950 : Tests, Debugging, Dev, etc.

### 902 :Command line options (tested)

Show command line options \[or use -h], but include all available and as yet unused single letter options \[a-z,A-Z] No options required

_Example: _python manage/cloader.py -u 902

### 904: Show Export Settings (tested)

Display the locations for all export types, and their current contents

_Example: _python manage/cloader.py -u 904

### 910: Loader Rules

A dump of the 'loader object', detailing loader rules and requirements. No options required. Prints as a dict of dicts without formatting.

_Example: _python manage/cloader.py -u 910

### 911: Header definitions

Print dataframe Header definitions. No options required. Prints neatly formatted.

_Example: _python manage/cloader.py -u 911

### 913: Get TID (May be buggy)

Testing for Target Dictionary lookup. Tests are failing on 137, unsure why.&#x20;

_Example:_ python manage/cloader.py -u 913 -d 137

### 925: Compare tables (tested)

Compare two tables in two databases.&#x20;

* Table names defined in a comma separated string in '-t'(ie: 'table1,table2').&#x20;
* Database connections for two tables are defined with '-d' and '-y' resp. ie: -d = database1, -y = database2

_Example: _python manage/cloader.py -u 925 -d 101 -y 137 -t "ASSAY, DEPOSIT\_JOB"

### 926: Compare list of tables (tested)

Compare list of two tables in two databases.&#x20;

* Table names defined in a hard coded list.&#x20;
* Database connections for pairs of tables are defined with '-d' and '-y' resp. ie: -d = database1, -y = database2

_Example: _python manage/cloader.py -u 950 -d 137

### 927: Print DB list (tested)

List the databases described in dbconnect.py. Give their numeric ID, descriptive name and connection string.

### 950: Test for 'checkStructure': (tested)

Testing of code relating to the 'checkStructure' web service.

_Example: _python manage/cloader.py -u 950 -d 137

Example output for all of these can be found at /nfs/panda/chembl/chembl\_loader/cloader\_example\_output/
