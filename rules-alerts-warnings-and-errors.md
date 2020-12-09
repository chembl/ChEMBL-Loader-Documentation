# Rules: Alerts, Warnings and Errors

Rules are applied during the load process, and other processes controlled by the laoder application, to test whether the data conforms to certain expectations. Each rule has associated with it a single 'Penalty Score' value. The Penalty Scores range from 0 to 9 inclusive. The higher the score, the more serious the problem. Depending upon the Penalty Scores accrued during an attempted load, one or more of the deposition load files may be assigned the status of 'invalid'. If one or more files in the load are 'invalid' then the entire load is aborted. Rules may either be applied to each individual record in a file \(eg: 'Field X must contain an integer'\), or, to the file as a whole \(eg: 'The filename must have a particular spelling or case', or 'field names must all match a list of permitted field names'\).

### Critical Errors

The highest Penalty Scores are termed 'Critical Errors'. 

* A Score greater than or equal to the 'Threshold for Critical Errors' is classified as a 'Critical Error'. 
* One or more  'Critical Errors' associated with a file will render the file invalid.
*  By default, the 'Threshold for Critical Errors'is set to a value of 7.
* The Threshold may be manually adjusted for each job at load time \(by use of the '-T' command line option\)
* The Threshold for Critical Errors must be no larger than 9, and must not be greater than the 'Threshold for Alerts becoming Warnings' \(see below\).

### Alerts

* Penalty scores greater than 0, up to and including the 'Threshold for Alerts becoming Warnings' are termed 'Alerts'.
* These have no effect on loading but are considered to be worthy of being logged. 
* The 'default Threshold for Alerts becoming Warnings is 2
* This value may be adjusted using the -H command line option from \(and including\) 1 up to \(but NOT including\) the 'Threshold for Critical Errors' \(see above\).

### Warnings

* Penalty scores greater than the 'Threshold for Alerts becoming Warnings' but less than the 'Threshold for Critical Errors' are termed 'Warnings'. 
* There is a Cumulative Warnings Threshold \(default = 20\). If more than this percentage of records in a file contain Warnings, then the file is marked as invalid. 
* For rules applied line by line, The Cumulative Warnings Threshold default value is 20. This value may be adjusted using the -W command line option to anywhere between 0 and 100. 
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

