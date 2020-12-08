# Running The Loader

## Running The Loader

The 'loader' is a application within the Gateway django project which can be run on the command line as well as via the Gateway interface. On the command line, many utilities can be run, but in the Gateway interface only the loading utility \(utility 1, -u 1\), can be run. Many options are available to alter the exact way in which the loader is run on the command line. Most of these options are available in the Gateway interface too for util 1. Below is described the basic operation of the command line loader, and some of the options available.

## Basic Operation: selecting a utility.

On the command line, '-h' will give a full list of available options, and '-l' will list all available utilities. A utility is run by using the '-u' option followed by an integer. Loading data uses util 1 \('-u 1'\). On the gateway interface, only -u 1 is available, and is preselected so the user need not do this.

## help

The '-h' option details all command line options available.

## Rules: Alerts, Warnings and Errors

Rules are applied during the load process, and other processes controlled by the laoder application, to test whether the data conforms to certain expectations. Each rule has associated with it a single 'Penalty Score' value. The Penalty Scores range from 0 to 9 inclusive. The higher the score, the more serious the problem. Depending upon the Penalty Scores accrued during an attempted load, one or more of the deposition load files may be assigned the status of 'invalid'. If one or more files in the load are 'invalid' then the entire load is aborted. Rules may either be applied to each individual record in a file \(eg: 'Field X must contain an integer'\), or, to the file as a whole \(eg: 'The filename must have a particular spelling or case', or 'field names must all match a list of permitted field names'\).

### Critical Errors

The highest Penalty Scores are termed 'Critical Errors'. 

* A Score greater than or equal to the 'Threshold for Critical Errors' is classified as a 'Critical Error'. 
* One or more  'Critical Errors' associated with a file will render the file invalid.
*  By default, the 'Threshold for Critical Errors'is set to a value of 7.
* The Threshold may be manually adjusted for each job at load time \(by use of the '-T' command line option\)
* The Threshold for Critical Errors must be no larger than 9, and must not be greater than the 'Threshold for Alerts becoming Warnings' \(see below\).

### Alerts

* Penalty scores greater than 0, upto and including the 'Threshold for Alerts becoming Warnings' are termed 'Alerts'.
* These have no effect on loading but are considered to be worthy of being logged. 
* The 'default Threshold for Alerts becoming Warnings is 2
* This value may be adjusted using the -H command line option from \(and including\) 1 up to \(but NOT including\) the 'Threshold for Critical Errors' \(see above\).

### Warnings

* Penalty scores greater than the 'Threshold for Alerts becoming Warnings' but less than the 'Threshold for Critical Errors' are termed 'Warnings'. 
* If more than a certain percentage \(Known as the 'Cumulative Warnings Threshold'\) of records in a file contain Warnings, then the file is marked as invalid. 
* For ules applied line by line, The Cumulative Warnings Threshold default value is 20. This value may be adjusted using the -W command line option to anywhere between 0 and 100. 
* For rules applied to files as a whole, a Warning behaves as an Alert \(ie: is not included in the cumulative warning calculation for a file\). 
* Note that a Penalty Score of 0 represents a 'Rule OFF', as it has no effect at all on loading \(like an Alert\), and is not even logged for the users attention. Thus, editing a Penalty Score for a Rule to 0 is an effective way of 'turning off' the rule without having to alter the code implementing the rule.
* If the Threshold for Alerts becoming Warnings is 1 less than the Threshold for Critical Errors, NONE of the Penalty Scores are treated as Warnings.

### Specific Penalty Scores

**Fatal Error Scores \(9\)**

* Penalty Scores of 9 are usually associated with rules which ensure data integrity, eg: a CIDX in an ACTIVITY file is not defined elsewhere in the load or in the DB.
* Data with these sorts of errors must never be loaded under any circumstances. 
* Penalty Scores of 9 can never be adjusted to 'Warnings' or 'Alerts' because the 'Threshold for Critical Errors' can never be set to greater than '9'. 
* The '9' Penalty Score is termed a 'Fatal Error' Penalty Score ,as it will always lead to an invalid file no matter te penalty thresholds.

**Structure Checks \(6+\)**

* Penalty Scores of '6' or above, have additional meaning in the context of 'checking structures': 
* Regardless of whether a score &gt;= 6 is classified as an Alert, Warning or Error, a score of &gt;=6 against a deposited structure in the COMPOUND\_CTAB file indicates that this particular CTAB should not be used during the compound Normalization procedures.
* Instead, the 'structure' should be assigned an 'empty structure' molregno.

**Error log level, and loader script termination**

During the load process \(which may be lengthy\), any Penalty Scores of '9' are sent to STDERR to alert the user that 'Fatal Errors' have already been detected. The '-U' command line option may be set to any number between 1 and 8 to give similar ongoing STDERR output for any Penalty Scores equal to or greater than the -U value.

Towards the end of the load process, when all deposited data has been processed and Penalty Scores are shown to the user, the user is told whether the numbers of errors etc has exceeded the thresholds for an acceptable data set to load. If ANY '9's were detected then the job will automatcially terminate. However, if no '9's were detected, but the load still did not meet quality criteria, then the user will be asked if they wish to attempt to load anyway. The default answer for this user question is 'no' \(so if '-a' is used, such data will not proceed to loading\). If the user answers 'yes', then the loader will attempt to load the data, and will reset '-H' to 8 and '-T' to 9 \(and record these values in the DEPOSIT\_JOB table against the current JOB\_ID\).

| PENALTY SCORE | CLASS OF PENALTY SCORE | SHOW IN LOAD REPORT? | LOG TO DB ON SUCCESSFUL LOAD? | COMMENTS |
| :--- | :--- | :--- | :--- | :--- |
| 0 | Rule OFF | No | No | Rule 'ignorred', or 'switched off' |
| 1 ...to... The 'Threshold for Alerts becoming Warnings' \[ default = '2', alter with -H \] | Alerts | Yes | Yes | No effect on loading. |
| ... to ... | Warnings | Yes | Yes | The 'Cumulative Warnings Threshold' \[ default = '20'\] The % records in a file with 1 or more warnings, above which a 'Critical Errors' is triggered. |
| The'Threshold for Critical Errors' \[ default = '7', alter with -T \] ... to ... 9 \[ ... a 'fatal error' \] | Critical Errors | Yes | - | Loading aborted if 1 or more such scores incurred. |

## Loading ‘Modes’ \(-m option\)

The loading utility \(-u 1\) is executable in different ‘modes’ \(set with –m option, \[1 unless specified\]\), as shown below. Four modes are available \(1-4\). Selecting a particular mode will run all modes up to the selected number \(ie: selecting ‘-m 3’, runs 1, then 2, then 3, then stops\).

| Mode | Description |
| :--- | :--- |
| 1 | File Validation \[simply run checks on the deposited set of files\] |
| 2 | Update report \[A report on what will be updated, whats new, etc. For updates, specifically which fields will be changed or remain the same\] |
| 3 | Update report \[A report on how ChEMBL will be updated. Eg: How many new assays, targets, etc\] |
| 4 | Loading. \[Load to the database. Only mode which triggers a new job\_id\] |

Note that all modes require a DB connection except mode 1. If mode 1 is run without a DB connection then data integrity checks can only be run on deposited sets \(so penalty scores are set lower if integrity rules are broken, as the missing data may exist within the DB\).

## verbosity \(-v\) and De-bugging etc.

All output from the loader is managed by the function 'verb', the second argument of which is called the 'level' \(an integer\). A verb call with a level of '0' always gives output. These calls are reserved for information such as 'loader started' etc. Levels from 1 - 9 are reserved for similar such programme flow information of lesser importance \(the higher the level, the less important the info is generally considered\).

The verbosity of the output is set with the '-v' option \(an integer\). The 'verb' function will give output if '-v' is greater than or equal to the 'level' for the function.

Levels higher than 9 are reserved for calls which output particular data structures, or related information on the content flowing through the program. These can be very useful during debugging. The levels assigned to such information progrssively inclrease throughout the program. So, for example, levels up to 1000 are used for 'mode 1' operations, levels 1000-2000 are reserved for 'mode 2' operations, and so on. These data can, of course, be outputted by raising '-v' to the level required, but this will also output all levels from 0 upto this raised level. Therefore, a more sensible way of viewing such structure is to use the '-w' or '-j' option. These two options have identical effects. Setting one of these to a particular level, and not setting the other, will output all calls with levels equal to this option. Setting both -w and -j to different levels will output all verb calls inbetween \(and including\) these two level values. This can be useful if a number of data structures are required to be viewed at once.

A special case for using -w and -j can be invoked when both are set to the SAME value. In these circumstances, all calls with levels equal to or higher than this value will be outputted, with the 'value' of the data structure or related info not shown, but instead 'subsituted' for the 'level' number. In addition, information on the calling subroutine of the verb function, and the data Type of the data structure is shown. This 'special case' is useful for identifying 'level' values that may be useful for a particular debugging process. In this special case, all '-v' levels are also outputted, but no 'substitution' occurs.

Here's some examples of how this works...

| option settings | 'verb' will output calls with these 'levels' \(inclusive\) |
| :--- | :--- |
| -v 6 -w 29 | 0-6 and 29 |
| -v 6 -j 29 | 0-6 and 29 |
| -v 3 -w 29 -j 1200 | 0-3, and 29-1200 |
| -v 3 -j 29 -w 1200 | 0-3, and 29-1200 |
| -v 2 | 0-2 0 |
| -j 29 -w 29 | &gt;= 29 \(but no 'values'\) |

Another useful option used in debugging is the option '-b'. Using this will append 'level' and timestamp information to the single record output of verb.

## Skip undefined CIDXs \(-Skip\)

SD files \(like the COMPOUND\_CTAB deposition file\), in contrast to tsv files \(like all other deposition files\), can sometimes be awkward to edit and parse for some users. For this reason, an additional '-Skip' option has been added to the loader, which enables undefined CIDXs to be Skipped \(ie: ignored\) during loading.

Under normal operation, the loader will issue a fatal error \(penalty score = 9\) if a CIDX in the COMPOUND\_CTAB does not exist in either the accompanying COMPOUND\_RECORD file or in the database. This ensures data integrity, and signals to depositors that a CIDX that they believe they already 'own', is, in fact undefined.

To alter this normal behaviour of the loader, the '-Skip' option may be used. This option accepts either a '1' or a '2'. These two possibilities have slightly different meaningings, which administrators should be aware of...

1 = Skip all CIDXs in COMPOUND\_CTAB deposition file that are ABSENT from the accompanying CR \(COMPOUND\_RECORD\) file, regardless of whether they are in the DB or not.

2 = Skip all CIDXs in COMPOUND\_CTAB deposition file that are ABSENT from the accompanying CR \(COMPOUND\_RECORD\) file AND ABSENT from the database.

## LoadTypes \(-L\)

When loading, the ChEMBL administrator may have a particular understanding or expectation of the result of loading a particular job, based on their knowledge of the nature and load history of the source. Thus, for example, some sources might be considered to usually only provide data where the load would not be expected to update any entities previously deposited in the DB \(but to simply incrementally add more entities\), and for the referential integrity to be fully defined within the job itself \(ie: with no reference to previously deposited entities\). An example of such a source would be molcon literature data, where each load is usually from a new collection of research papers that have not been loaded before, and where 'up dates' to previously deposited data are rare. Sources such as these are termed 'Cumulative Load with job-defined referential integrity', or LOADTYPE '2'.

When a a source is first created, a LOADTYPE may be defined in the SOURCES table. This may be null, or an integer defined as a PK of the SOURCE\_LOADTYPE table.

At load time, the value of the LOADTYPE for the source is obtained from the SOURCE\_LOADTYPE table, and the weighting of warnings and Errors are adjusted accordingly, in order to highlight and emphasize deviations from the differently anticipated result of the load. If a 'null' exists for the source, then the default LOADTYPE \(currently set as 1 in globalVars.py\) is used instead. If the command line option '-L' is used \(which takes an integer\) then this overides any of the above.

A LOADTYPE of '1' is considered the norm, but With LOADTYPE set to '2', the following changes are made...

* If any imminent 'Overwrites' to existing DB entities are detected, then a fatal error is assigned \(these would only be minor alerts with '-L' set to '1'\). Note: an exception to this rule is COMPOUND\_CTAB.sdf \(see below\)\].
* No 'advanced' checks for better altenative CR CRIDX combinations are carried out \(routinely done for -L = 1, but computationally expensive for large sets\).
* If there are CIDXs in COMPOUND\_CTAB.sdf which are not present in COMPOUND\_RECORD, then simply ignor them in subsequent processing \[ie: do not allow them to overwite any existing CIDX structures that may exist\], AND demote the errors normally produced in these circumstances \(ie: if the CIDX does not exist in the DB\), to a minor alert.

Currently, only 2 LoadTypes are recognized, but others may be added in future, if required.





