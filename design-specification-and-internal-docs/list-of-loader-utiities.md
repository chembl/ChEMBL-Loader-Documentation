---
description: List of loader utilities for cloader.py
---

# List of Loader Utilities

## 1 to 20 : Process deposition data 

### 1: Load

Process deposition data. Requires -m, -L \[-d\]

### 2: Correct \[CURRENTLY UNAVAILABLE\]

Make a correction to some deposited data. Only 1 file at a time .. update ONLY fields shown.Ie: just TITLE to correct a spelling error in the title ? IGNORE all null or empty fields.

### 10: Compound Normalization:

Normalize recently loaded compounds. The normal operation for the 'Compound Normalization' utility in the loader is to:

* First run a DB query to identify all CIDX/SRCID combinations requiring normalization. 
* Then, process these in batches of 10 through a normalization process and commit at the end of each batch. 
  * The commit step requires a response from the user, although if the '-a' option is used then the default is used instead \('yes, commit' in this case\). 
  * Processing of batches will continue until all CIDX/SRC\_IDs have been normalized.

**Options**

* If the user wishes to change the batch size from 10, then they can do this with the '-y' \(lower case\) option \(which takes an integer\). 
* If the user wishes to limit the total number of CIDX/SRC\_IDs processed in one run of the script, then this may be altered in a similar way by using the '-Y' option \(uppercase\). 

Thus, for example, running with the options ' -y 20 -Y 502 ' will result in the processing of 502 CIDX/SRC\_IDs, 20 at a time \(ie: 26 batches, with 20 CIDX/SRC\_IDs in all but the last batch, which would contain 2 CIDX/SRC\_IDs\).

**Webservice requirements**

* The standardization process included in the normalization process requires the calling of the ChEMBL RDKit web-service. 
* If a '500' error is returned during any single batch then the normalization r un is aborted. 
* Setting -Y and -y to reasonably low numbers \(and perhaps stringing together multiple script runs consecutively on the LSF\) mitigates the risk of network interruptions resulting in the 'loss' of large numbers of normalizations before they can be committed. 
* However, the cost of setting -Y and -y too low, is that: 

  * 1. Each run of the Normalization process requires a full query o f the CR table to assess which CIDX/SRC\_IDs currently require normalization**.**
  * 2. Each batch req uires a single run of the InChI generation software \(it is far more efficient to run this for multiple structures simultaneously.\)

### 14: Scan for Cleaner Structures \(tested\)

Run a search for deposited structures which have lower penalty scores \(from the 'structure checker' web service\) than the current ChEMBL CTAB for the same InChIKey

* Requires -J

### 17: Delete Activities

Delete all 'Activities' data relating to a given LIST of EITHER

1. AIDX's from file '-f'
2. 2.JOB\_IDs on command line \('-J'\). '-s'

The owner of the AIDX's or JOB\_ID's, must also be specified. 

* Note that ALL related SUPPLEMENTARY and PROPERTY data is also deleted, as is all corresponding data in 'DEP' tables. 
* Tables affected: ACTIVITIES, ACTIVITY\_PROPERTIES, ACTIVITY\_SUPP, ACTIVITY\_SUPP\_MAP, ACTIVITY\_SMID, DEP\_ACTIVITY, DEP\_ACTIVITY\_PROPERTIES, DEP\_ACTIVITY\_SUPP, DEP\_ACTIVITY\_SUPP\_MAP 
* Note also that certain ChEMBL tables are connected to 'parent' tables with a 'cascade delete' in place, so are automatically deleted if parent PKs are deleted. Thus, 'ACTIVITY\_SUPP\_MAP' and 'ACTIVITY\_RGID\_MAP' are cascade connected to 'ACTIVITY\_SMID', and 'ACTIVITY\_PROPERTIES' to 'ACTIVITY'.

## 21 to 40 : Job management

### 21: Show jobs \(tested\)

Show all jobs. Filter to show only for one source if '-s' option used. Requires -d. add -v for all fields. Use -J to limit to a range of JOBIDs

### 22: Report job\(s\) in sets \(tested\)

Show all the jobs, and the sets they belong to, for either a single job \(as -J int\), a list of jobs \(as -J in a hyphen separated list\), or a single set \(as -Set int\). Requires -J or -Set. Ifquerying with a single set, then CHEMBLID may be used instead of set\_id. Note that in the resultsshown, a single job will be listed more than once if it belongs to muliple sets.

### 23: Export job\(s\) \(tested\)

Export all data for either a single job \(as -J int\), a list of jobs \(as -J in a hyphen separated list\) or a single set \(as -Set int\). If single set, then CHEMBLID may be used instead of set\_id. Requires -J or -Set, and -O \[a valid directory, or, to print to screen, use one of either '-','none' or 'screen'\]. If separate files required for separate jobs, then use the -M option, to create subdirs named 'jobID\d+' in -O

### 24: Show job load report \(tested\)

Show the report on a load for a single JOB\_ID \(-J\). Requires -d

### 25: Show content report \(Jobs or AIDXs\)

Show a report of the content assigned to JOB\_ID\(s\) \(-J, eg: -J '1-4,6,8-9'\), or a list of AIDXs present in a text file, location '-f'. Requires -d, -s

## 41 to 60 : Set management 

### 41: Create a set

Create a set. Requires -d and -J \(hyphen separated list of Job\_ids\)

### 42: Edit a set

Add or Remove a single JOB\_ID to/from an existing set. Requires -d, -J and -Set

### 51: Show sets 

Show all sets. Optionally, use -s \[int\] to filter and show data for only one source. Requires-d \[-s\]

### 52: Flag a set as public \[NOT FULLY TESTED\]

Switch the 'Public' flag to '1' for this set, so that it may be Published in the future. Requires -Set \[int\] and -d

### 53: Freeze a set \[NOT FULLY TESTED\]

Freeze an existing set. Requires -Set \[int\] and -d

### 54: Thaw a set \[CURRENTLY UNAVAILABLE\]

Thaw a frozen set. Requires -Set \[int\] and -d

### 55: Export a set  \[NOT FULLY TESTED\]

 Export a set. Destination determined by public and freeze flags for particular set. Requires -Set \[int\] and -d.

## 81 to 120 : Schedules 

### 81: Run a schedule

Run a schedule. Requires -d and -N82= Show a schedule... Show a schedule. Requires -d and -N

## 121 to 140 : Sources 

### 121 Create a new Source:

 Create a new SRC\_ID. Requires -d, -s \(new src\_id\), -n \(new name\), -Z \(new description\),, -y \(year\)

## 141 to 160 : Processing, Analysis, etc. 

### 141: Get Activity Data \[NOT FULLY TESTED\]

Print Activity data for a list of Activity\_ids or a Job\_ids \(Requires -d, -p and -J or -A\).

* '-p' is a pipe separated property list with 3 elements \(0-2\), each used to indicate what should be included \(eg: '-p APM\|IN\|T'\)
  * 0 = 'Which Tables to Query'
    * A = ACTIVITIES 
    * P = ACTIVITY\_PROPERTIES \(Shown separately with 'T' but merged onto A with 'M'\)
    * M = ACTIVITY\_SUPP\_MAP \(Only shown separately for 'T' option\).
    * E = ACTIVITY\_SMID \(Only shown separately for 'T' option\)
    * S = ACTIVITY\_SUPP, Excluding records with null SMIDs
    * U = ACTIVITY\_SUPP, but Include records with nUll SMIDs. Only affects 'M'.
    * Note: No P without A, and no U without S.
  * 1 = 'Which Columns and Cells to Show'
    * \[NB: These do not apply to 'T' option below\]
    * I = Identifiers \(Include SMIDs\).
    * r = show raw \(pre-normalized\) data.
    * R = show raw \(pre-normalized\) data, and pivot on these data.
    * n = show normalized form of data.
    * N = show normalized form of data, and pivot on these data.
    * Z = Show Full SMID data from supp in ACT matrix \[otherwise just show SMID values if S selected above\].
    * C = Compact. If the values in a 'units' or 'relation' column are all the same, then put these into the 'type' header, and omit the column from the output.
    * F = Fill pivot gaps. Fill the gaps you'd normally have in pivot table with appropriate data.
    * t = transpose table \(by 90 degrees : headers become row names\)
  * 2 = 'Output Format'.\[NB: choose either M or T\]
    * T = Tables, as per database, nothing added or subtracted and no format changes.
    * M = Matrix \(2D\). Make 3 matrices: 'ACT', 'PROP' and SUPP' 2D matrices. If no numbers, then make ALL, else make only numbered ones.
      * 1. 'ACT' matrix is a P and S appended as extra cols onto A.
      * 2. 'PROP' matrix is a pivot of P onto A with Types as headers, using ACTIVITY\_IDs as row numbers.
      * 3. 'SUPP' matrix is a pivot of S/U alone with Types as headers, using RGID as row numbers. Will be created from S or U, whichever is selected.
  * Data is printed to console by default. 
    * Use '-y' \[int\] to constrain col spacing, '99' to autofit.
    * Use '-Y' \[int\] to constrain col1 spacing \(best for transformed data only\).
    * Use '-Z' \[str\] show only these cols. eg: -Z '2-5\|8\|11\|14\|17-20\|23-28\|34'.
    * First col is '0'. Show all cols if -Z not used.
    * Use '-X' \[boo\] to truncate printing to console so lines do not overrun to the next line.
    * Use '-Q' \[str\] for an alternative to '\|' as a separator for 'Identifiers' \(see above\) if multiple exist.
    * Use '-O' \[str\] for printing to file instead of console \[give directory \('.' if current dir\)\].
    * Use '-z' \[int\] to limit all queries using 'rownum &lt; z' \[useful to see T structure\]
  * Comparisons to original data may be required
    * Use '-k' \[str\] path to 'kLol' \(a picKled 'List of lists' \[usually captured spreadsheet\]\) data. Show as a separate matrix after matrix 3.
    * Use '-K' \[boo\] merge M3U \(Matrix 3 using 'U' query\) INTO a copy of kLol\(from '-k' above\), and show as another separate matrix \(requires '-k'\).
    * Note : it is inadvisable to select AS and Z unless 'G' is true \[ie: Full Granular Mapping curation has been carried out\].Otherwise, if very many A records are each mapped many or all S records, then theACT matrix could be very large indeed, as there will be a separate row for each A -to- S combination.

## 901 to 950 : Tests, Debugging, Dev, etc.

###  902 :Command line options

Show command line options \[or use -h\], but include all available and as yet unused single letter options \[a-z,A-Z\] No options required

### 903: Confirm synch \[CURRENTLY UNAVAILABLE\]:

 Confirm that code is sychronized with the DB. Requires -d

### 90: Show Export Settings

Display the locations for all export types, and their current contents

### 908: Print Job Supporting Data \[NOT FULLY TESTED\]

Print to the console a 2D-matrix of all supporting data \(from ACTIVITYSUPP\) that is associated with a Job \(Requires -d and -J\)

### 910: Loader Rules

A dump of the 'loader object', detailing loader rules and requirements. No options required

### 911: Header definitions

Print dataframe Header definitions. No options required

### 912: UC Activities for job

Normalize' Activity, Activity\_supp and Activity\_property data 

* Use -p 'APS\|\|' to update all3\] for a given \[-J\] JOB\_ID, by UPPERCASING everything - this is just a debug method to help generate some normalized fields quickly - DONT EVER RUN ON PRODUCTION !!!! 
* No options required

### 913: Get TID

Testing for Target Dictionary lookup.

### 921: Migrate Source \[IN DEV. DO NOT USE. NOT FULLY TESTED\]

* Migrate all data for a source into new loader format. 

### 925: Compare tables

Compare two tables in two databases. 

* Table names defined in a comma separated string in '-t'\(ie: 'table1,table2'\). 
* Database connections for two tables are defined with '-d' and '-y' resp. ie: -d = database1, -y = database2

### 926: Compare list of tables

Compare list of two tables in two databases. 

* Table names defined in a hard coded list. 
* Database connections for pairs of tables are defined with '-d' and '-y' resp. ie: -d = database1, -y = database2

### 931:DELETE DEP data \[IN DEV. DO NOT USE\]

Delete all data in DEP\_tables for a given source \('-s'\). 

* CARE over use of this. 
* Used when developing 'migrate source'. 

### 949: Experimental code

Testing of code snippets

### 950: Test for 'checkStructure':

Testing of code relating to the 'checkStructure' web service.

