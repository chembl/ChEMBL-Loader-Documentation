# Complex Results Sets

ChEMBL has always handled simple forms of bioactivity data extremely well, such as 'cpdX has affinity \(Ki\) of Y uM in assay Z.' \(and perhaps with a reference to the source of this data, to indicate where the raw or supporting data may be found\). Such data is represented by a single record in the ACTIVITY load file, such those shown in 'Activities' example in the Examples link above. For most depositors, this data model is quite sufficient to capture their data. Such depositors need only deposit their data using the ACTIVITY deposition file, and read no further.

However, there is a growing need for more complex data to be captured within ChEMBL. Such data may come from assays where multiple result types are required to describe the activity of the test compound \(ie: not just a Ki value, as above\), or where different Result values of the same result Type must be 'qualified' or 'contextualized' to convey the result faithfully. In other cases, it may be important to provide supporting or supplementary data, and not simply a reference to a literature paper.

Here, we describe a model for how ChEMBL will deal with Complex BioActivity Result sets in the future. The new model has been designed with the following requirements in mind. The new model must...

1. Be appended to the existing model for simpler data \(ie: no re-working of existing data should be required, and no effect on existing users who are quite happy with the existing model and the data it holds\).
2. Be as simple as possible, so that the barrier to using it is as low as possible, for administrators and depositors alike.
3. Be capable of handling _any_ kind of Complex BioActivity Result Set, to any degree of required granularity.
4. Maximize the degree with which the data can become integrated within ChEMBL as a whole.

To describe the model, we break the discussion below down into 'Multiple Result Types', 'Properties', and 'Supplementary data', and consider them separately. Note, however, that in practice many complex assays require all three to be handled simultaneously.

Since it is often easier to understand these complexities by reference to examples, the reader should note that examples for all that follows are given under the 'Complex result Set' category in the 'examples' link above.

Note also that because the model can \(necessarily, unfortunately\) become quite complicated to work with, many of the 'queries' required to make use of the data can be 'canned' into more manageable forms for users less familiar with the details of how data is actually stored.

## Multiple Result Types

Many result sets contain multiple Result Types of high importance, and often a combination of these Result Types is used to judge the outcome of the assay. An example might be potency and efficacy in a functional assay, which might be measured with Result Types such as EC50 and IA, respectively. Other assays speculatively measure very many result types simultaneously; a signal from any one of which would suggest a 'signal' in the assay. An example might be the measurement of multiple markers in the blood after dosing with a particular drug.

In the traditional ChEMBL model, each of these multiple Result Type values would form a separate record in the ACTIVITY table. In the simple case where a compound has been tested once in an assay, and two result types are reported \(say, an EC50, and a corresponding IA\) then the user may infer that the different result type values may be related \(ie: the EC50 and the IA were 'read' from the same curve\). This inference may be fair, but is not formalized: there is no formal relationship in the database that relates the two result types to one another. Now consider the case where the compound has been tested twice: the user cannot tell which of the two EC50s for the compound are related to which of the two IAs.

In other words, the connection between multiple result types for an assay are not formalized, and if multiple tests of the same CIDX are present, then this connection is ambiguous.

To formalize these connections, and remove any ambiguity from the data, the 'TEOID' \(TEst Occasion ID\) field is used to tie together Result Type values that were obtained on the same 'test occassion'. The TEOID is an optional field, but if it is not empty then it must contain a positive integer. The TEOID only has meaning within the deposition set \(ie: the job\), so values \(such as 1,2,3, etc\) can be re-used between depositions.

## Properties

Suppose that an assay is used to generate Result Values of a particular type, but the values quoted must be qualified with another parameter that has been measured or set by the Assay. An example of this might be an inhibition experiment where 'slope' and 'goodness of fit' information may be attached as a property, as in the example below. Such property values are often useful in assessing the mechanism behind the result, or the quality or reliability of the reported main value.

These Property data are captured and stored in an ACTIVITY\_PROPERTY file/table. Records in the ACTIVITY and ACTIVITY\_PROPERTY files must be tied together using the ACT\_ID, which must always be a positive integer, and only has meaning within the load job \(so 1,2,3, etc, can be re-used between load jobs\). In the ACTIVITY file, the ACT\_ID must be unique for each record, although not all records need have a value \(since some Activities may not have properties associated with them\). In the ACTIVITY\_PROPERTY file, each ACT\_ID can occur multiple times, but must match an ACT\_ID in the ACTIVITY file in the same deposition. All records in ACTIVITY\_PROPERTY must contain an ACT\_ID.

Sometimes, it is debatable as to whether a result \(such as slope, for example\) should be considered a property \(in ACTIVITY\_PROPERTY\) or a result type \(in ACTIVITY\) in its own right. There is no hard and fast rule to make this distinction, but as a rule of thumb, the Activity values should be the main variable mesures in the assay, and ACTIVITY properties should be derived data or data specific to the single assay in question.

## Supplementary data

'Reference ID's \(RIDXs\), which equate to Document IDs in ChEMBL have traditionally been used as a way of pointing users to supplementary and supporting data located externally to ChEMBL. This mechanism still exists, but it is also now possible to deposit such additional supportive or supplementary data.

An example of this \(as shown in the 'Examples' link above\) might be the individual %Inh points on a curve where an IC50 value has been quoted as an Activity. Supporting data such as these are stored in the ACTIVITY\_SUPP table, but require the introduction of two new identifiers: REGID and SAMID.

### REGID and SAMID identifiers.

A **REGID** identifier \(REcord grouping ID\) is used to cluster or group together related records in the ACTIVITY\_SUPP file/table, just as the TEOID was used in the ACTIVITY table \(as explained above in 'Multiple Result Types'\). The REGID is a required field, and must contain a positive integer. Also, it only has meaning within the deposition set \(ie: the job\), so values \(such as 1,2,3, etc\) can be re-used between depositions. When preparing data for loading it can be useful to think of REGIDs as 'row numbers' in a 2D table of data \(see below for more on this\).

An additional table \(ACTIVITY\_SUPP\_MAP\) uses **SAMID** identifiers to map which records in the ACTIVITY\_SUPP table are supporting evidence for which records in the ACTIVITY table. SAMIDs are mandatory in the ACTIVITY\_SUPP\_MAP file/table, but the SAMID field in ACTIVITY\_SUPP can be left as null if required \(ie: if the Supplementary record does not directly point to any Activity value in particular - see later for an example\).

### Examples

To understand more clearly how REGID and SAMID ids are used, please study the 'Complex Result Sets: Supplementary Data' examples in the 'Examples' link above, alongside the text below.

In the simplest example shown, note how the reported IC50s each have separate sets of supporting evidence in the ACTIVITY\_SUPP table. The 'Test conc' associated with each '%Inh' is indicated by use of a shared REGID.

Now consider a more complicated example, where the reported IC50 values each have some separate supporting data records, but _also_ have some supporting data records _in common_. In this case, REGIDs are used in much the same way as before, but the SAMID is used to correctly point each Activity to both it's own separate supporting data sets, _and_ the shared supporting data records. This is useful in the situation where an assay has made use of shared controls.

Now consider an example where the ACTIVITY\_SUPP table contains some records which are considered only indirectly relevant to particular ACTIVITY values \(see the 'null SMIDs' example\). Such data might be included in the supporting data set as they may be considered of use to users who may wish to study the very fine details of the raw data. Examples of this might be raw counts from a reader, etc. In these cases a SAMID mapping to the ACTIVITY value is _not_ required \(ie: a 'null' should be used in the SAMID field of the ACTIVITY\_SUPP table/file\).

However, these 'null' SAMID records must be tied in some way to the relevant ACTIVITY\_SUPP records which _do_ have SAMIDs. This is achieved with the REGID. Thus records with null SAMIDs must therefore _share_ a REGID with another record that has a non-null SAMID \(see Integrity Constraints below\).

The querier of these data may then perform two different types of queries, depending on how much of the relevant data they require. A query using only SAMIDs will retrieve only the most directly related data supporting an Activity record. However if _all_ supporting data is required then a sub-query which joins the ACTIVITY\_SUPP table to itself using REGIDs is required.

### Integrity Constraints

The reader should now appreciate that in order for all data to be loaded correctly, the following integrity constraints must be imposed on the deposition files...

1. All SAMIDs used in a job must be present at least once in both ACTIVITY\_SUPP and ACTIVITY\_SUPP\_MAP
2. All ACTIVITY\_SUPP records where SAMID is 'null' must have a REGID that is also present in at least one ACTIVITY\_SUPP record where SAMID is non-null.

### Flexible SAMID mapping

For very large and complicated data sets, with multiple controls \(perhaps some shared, some not\), the creation of the correct SAMID mappings can be an onerous, and often error-prone, curation task. This task can be a considerable barrier to submitting data. For this reason, SAMID use is interpreted flexibly. Thus, when submitting the supplementary data the _granularity_ of the SAMID mapping could be very high or very low \(depending upon how much curation effort is available\). The lowest, or 'default' mapping would be to simply map all Activity values for an assay in a job to all the supplementary data supplied. Using this lowest default mapping allows supplementary data to be loaded easily and quickly: high granularity mapping can be postponed until curation resources become available, and then the entire set updated at a later date. See the example 'Complex mapping deferred'.

Of course, data loaded with low granularity mapping has limited usefulness from the point of view of the user querying such data: the querier can identify the entire data set from where a given Activity came from, but unfortunately is left to guess exactly which ACTIVITY\_SUPP data values were used to generate which Activity values.

Some users may therefore require to be able to distinguish between high and low granularity mapping \(perhaps to filter out low granularity mapping\). For this reason, on loading to ChEMBL an option \('-G' on the command line loader\) exists to signal whether the supplementary data in the job has been mapped to Activities with the highest granularity possible. Depositors of Complex Data sets must state whether this option should be used \(it cannot be encoded within the deposition files\). If this option is not included, then by default the job is marked as 'incomplete SAMID mapping'. However, if the option is used, then the set is marked up as 'SAMID mapping completed', and this information is stored in a table called ACTIVITY\_SUPP\_META. This then allows the querier to determine \(by a lookup to this table\) whether the retrieved Supplementary data associated with a single Activity record in this data set will contain only data which directly supports the Activity value.

### Summary of the Advantages of the SAMID REGID model.

Although the loading of Supplementary data suffers from the shortcoming above, it is worth emphasizing some of the advantages of loading these data...

1. The barrier to loading Supplementary data is lowered \(The fine grained SAMID mapping curation of very complex data can be achieved at a later date when resources permit, but does not delay loading\).
2. Smaller, simpler supplementary data sets can often be 'SAMID mapped' quite simply, so the supplementary data becomes available for querying immediately in these cases.
3. The Supplementary data is normalized \(using the same algorithms applied to 'ACTIVITY' and 'ACTIVITY\_PROPERTY'\), maximizing the wider value of the data in the integrated whole of ChEMBL.
4. In many cases, using the REGID in a particular way \(see below\) enables the original 2D matrix of the complex data to be recreated simply \(and exported, together with normalized data\).

### Complex result Sets as a 2D matrix.

For many scientists one of the most convenient ways of browsing the values in a highly complex result set is in the form of a 2D matix, such as a speadsheet. Indeed, this is the most common format for complex result sets presented to ChEMBL administrators for loading into ChEMBL.

The ACTIVITY\_SUPP table can be thought of as a transformation of a 2D matrix where REGIDs represent rows, the 'TYPE's are the column headers, and the 'VALUE's are the cells. In fact, one of the best ways of creating the ACTIVITY\_SUPP table/file can be to start with a 2D matrix of all the data to be loaded, and transform these data into the ACTIVITY\_SUPP file/table with a script. By doing this, reconstructing the original data set \(ie: the 2D matrix\), _along with normalized versions of the data_, is straightforward when such data are subsequently exported from ChEMBL.

## Other Practicalities.

One of the practical considerations for us at ChEMBL is that we have fairly limited resources, so we will not always have time to get to grips with understanding extremely complex assays to the same level as the creators of the data. Most 'depositors' of complex result sets will probably ask us to reformat the data for loading. For this reason, our process may have to involve asking depositors to ...

1. Provide a 2D matrix \(speadsheet\) of the data to be loaded.
2. Identify the minimal self defining, self- referencing 'unit's or 'sets' within the complex data \(ie: all the controls and tests within the set can be used in a variety of ways within the set, and are not relevant outside the set\).
3. Identify the top-level result types from the set \(eg: % Apoptosis, and % Interphase \[more?\] in your case, perhaps\), and advise on how these should be averaged \(if at all\), and what 'properties' may be associated with these \(eg: test conc, etc\).
4. Advise on how the set\(s\) may be pivoted to one or more top level result types

