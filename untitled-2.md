# embargoAndTagging.rst

Tagging of a data set is a utility that enables a depositor to refer to a given set of their data within ChEMBL. The process of ‘tagging’ involves the creation of a SET\_ID. The SET\_ID contains one or more ‘job\_id’s. Each of the job\_id data sets within the SET\_ID may be edited, provided they are not shared with other SET\_IDs that have been frozen. In this un-frozen state the SET\_ID is termed ‘fluid’.

Freezing of a SET\_ID prevents a depositor from updating, deleting or inserting any data into a defined, deposited tagged set \(NB: Curated data associated with the deposited set may still be changed, as new rules and normalization procedures are edited and introduced\). Within ChEMBL, curation efforts often require that certain curated data be ‘locked’. The term ‘freeze’ is used here instead of ‘lock’ simply to try to make clear the difference between this same action on raw deposited data and curated data: ‘freeze’ is therefore used specifically to refer to the ‘locking’ of deposited, precurated data.

The addition of the DEP\_ tables provides a very clean and clear distinction between deposited and curated data, and therefore greatly simplifies the mechanism for freezing data, as only the rows in these tables must be ‘frozen’, as described below.

Tagging of data sets is a first step to ‘freezing’, with the option left open for the depositor to decide whether they would like to progress to freezing their data set in the future.

Making tagged and frozen data sets browsable in the release schema via the ChEMBL interface may be complicated to implement, and will always rely on the data only being available after the entire schema release process has run. This will severely restrict the timings for when frozen sets can be made available.

A far simpler and more flexible option, recommended here and described below, is that only simple downloads of the raw, precurated data are made available. Such downloads would be taken directly from the production schema, and so could easily be created any time.

Depositors would also be given the option to have their tagged set be either ‘public’ or ‘private’. Public sets would be exported to a public ftp site, but private sets would be exported only to private directories behind the EBI firewall, and then manually emailed to depositors by ChEMBL administrators. The download would consist very simply of several files \(sdf and txt\) which contain essentially the same data as was first deposited by the source, and nothing else, except for a README, and possibly an md5checksum file

Required Changes to the existing Schema

It is proposed that all DEP\_ tables \(but no other tables\) have a ‘FROZEN’ field added, and that freezing is reliably implemented by the addition of a trigger to each DEP\_ table that prevents any field \(including the ‘FROZEN’ field itself\) in the record from being edited if this field is ‘1’ \(example trigger in appendix G\). Freezing a record is therefore simply achieved by updating this field to ‘1’ from null or ‘0’. On the other hand, ‘Tagging’ of data does not require any edits to the DEP\_ tables.

The management of ‘Tagging’ and ‘Freezing’ will require two new tables in ChEMBL:

The ‘DEPOSIT\_SET’ table, with fields:

FIELDDatatypeDescription SET\_IDint not nullThe set id. PK CHEMBLIDvarchar2\(40\)The CHEMBLID for this set\_id DESCRIPTIONvarchar2\(4000\)Free text description of the set. SRC\_IDint not nullThe owner of the set\_id PUBLICchar\(1\) not null0=private, 1=public PUBLIC\_DATEdateThe date when ‘public’ changed to ‘1’ FROZENchar\(1\) not null0=fluid, 1=frozen FROZEN\_DATEdateThe date when set was frozen. LAST\_PRIVATE\_EXPORTdate Date when set was last exported as a private set. LAST\_PUBLISHED\_FLUIDdateDate when the set was last published as a fluid set \[always prior to ‘last\_published\_frozen’ if exists\] LAST\_PUBLISHED\_FROZENdateDate when the set was last published as a ‘frozen’ set. CREATEDdateDate when this record created UPDATEDdateDate when record last updated

The ‘DEPOSIT\_SET\_MAP’ table, with fields \(a composite PK \(SET\_ID and JOB\_ID\) is used\):

FIELDDatatypeDescription SET\_IDint not nullThe set id. PK JOB\_IDint not nullThe job id. PK FROZENchar\(1\) 0=fluid, 1=frozen CREATEDdateDate stamp

Code functions to manage sets.

In addition to functions to create edit, freeze and thaw sets, and switch on the public flag, a generic ‘export’ function or base class will be created. This exports all data associated with a list of job\_ids, to a single tarball \(Option not to tarball, and Option to keep files in separate dirs.\). A ‘README’ file produced by default, detailing status of data … public or private, frozen or fluid, export location, date, number of files, md5checksums ??? etc etc. Format of filename… CHEMBL64538.tar.gz or of directory if files kept separate… CHEMBL64538

Two functions may be used, which are wrappers around base class ‘Export’… 1.Private Export. First, search for embargoed data. If yes, list the sources who own these embargoed data. If more than one source, then big warning required. Then ask if you want to exclude the embargoed data ? Default export location to internal locn \(/nfs/etc ??\). but option to change this. 2.Publish. Function works with following workflow… Is it public ? if ‘no’, then exit. But if yes then… Is it frozen ? If yes… Does it have embargoed data ? If yes, then exit, but if ‘no’ then.. Run ‘export’ and default write to...

> ftp//ebi/.. /ChEMBL/publishedDataSets/frozen then, delete same named file, if it exists, in ‘fluid’ \[check whether it SHOULD exist by querying ‘LAST\_PUBLISHED\_FLUID’\]
>
> > . . . BUT. . if its NOT frozen \(still fluid\).. then… Strip out all job\_ids currently under embargo still, then . . Run ‘export’ and default write to…
>
> ftp//ebi/.. /ChEMBL/publishedDataSets/fluid

README’s in publishedDataSets/, fluid/, and frozen/. The README in the publishedDataSets/ must explain that only if the file modification time preceeds the most recent ChEMBL release can it be guaranteed that all the IDs within the download are present in this most recent ChEMBL release.

The README in the fluid/ dir must explain that the data content of any set in this directory may change over time. One reason for this may be that the owner of the job\_ids within this set may choose to update these data. Another reason may be that the set within ChEMBL contains data still under embargo. Only when a set is ‘frozen’ \(and no longer appears in this directory, but in the frozen directory instead\) can the user be assured that the set data will not change, and that all data is present \(ie: any data in the set that may have been under embargo at some point has now either been permanently removed from the set, or is included as it is no longer under embargo\).

How it works from the User’s perspective.

A depositor indicates that they wish to ‘tag’ a data set. This must be all the data from one or more job\_ids \(ie: it is not possible to ‘tag’ part of a job\_id\). The administrator then runs loader code to create a new ‘set\_id’ \(a single new record in DEPOSIT\_SET, associated with the job\_ids specified \(one new record per job\_id in DEPOSIT\_SET\_MAP\). A description would also be required at this stage. The tagged set may include ANY valid JOB\_ID \(ie: the JOB\_IDs may be embargoed, frozen, and from any depositor\). This tagged set is assigned a SET\_ID and a CHEMBL\_ID.

When creating a new SET, or adding new JOB\_IDs to an existing SET\_ID, warnings are given if any of the JOB\_IDs are …

1.Embargoed, or 2.Frozen, or 3.Do not all belong to the owner of the set, or 4.Any combination of the above.

These warnings are given because if any of the above occur certain limitations will exist on the subsequent use of the set. These limitations become clearer when exporting, publishing and freezing operations are carried out, as explained below.

If, at any stage, the depositor wishes to edit data in any non-frozen job\_id which belongs to them in a tagged set, they are free to do so. However, they cannot edit JOB\_IDs that are either frozen or do not belong to them.

If the owner of the tagged set chooses to ‘freeze’ the set, then this operation will only be successful if all the JOB\_IDs in the set are owned by the owner of the SET\_ID. This is to prevent a user freezing another depositor’s JOB\_IDs. The same operation will still fail even if these JOB\_IDs owned by another depositor are already frozen. This is prevented as in some very rare circumstances it may be necessary to ‘thaw’ a set or job\_id \(see later\).

Also, for the purposes of review for a publication \(either by the author or the reviewers\) it may be necessary to export, at anytime, a ‘tagged’ or ‘frozen’ data set prior to the production of a ‘frozen\_release’ download of this set. See ‘Private Export of a Tagged Data Set’ for more details on this.

Later, the depositor may choose to ‘freeze’ the previously ‘tagged’ set. The advantage of creating a frozen set is that users may cite the data set in publications knowing that the data cannot change. It also means that a defined download set can be created \(and hyperlinked to by external users, if required\). The frozen download also has the advantage that it can be created anytime, independently of the ChEMBL release cycle.

The freezing process is reversible, but administrators should not encourage or advertise this operation to depositors. Thawing is a difficult and potentially error-prone database operation, requiring the disabling and re-enabling of triggers, so should be avoided. Certainly, thawing is should be strictly forbidden if the set has been published. This is to ensure that ChEMBL users an have confidence that publically available ‘frozen’ sets are immutable with a stable ID. This can be important for publication purposes, where authors may cite frozen sets. Thawing of frozen, non-public sets is permitted, but again, not encouraged because of the awkward database operations required.

> and is implemented by the administrator, who runs the loader in a mode which sets the ‘FROZEN’ field, in all DEP\_ tables, to ‘1’ where the JOB\_ID is a part of the set\_id to be frozen.

On the ftp site, users would be able to find downloads for all public sets, listed by CHEMBL\_ID. Tagged \(or ‘Fluid’\) sets and Frozen would appear in separate directories to highlight the difference for users.

In the release schema interface, users and depositors would be able to view a “ ‘Tagged and Frozen’ data sets “ page which would simply have a link to the current FTP site, with warnings to take note of the file modification times \(as only those files with times prior to the ChMBL release date can be guaranteed to not contain IDs that are not in the current release\).

Public Release of a Frozen set.

As a nightly cron job, the loader would be run in a mode which would, in a single transaction:

1. Generate a list of all SET\_IDs where ‘Release’ = ‘0’ \(ie: data has not yet been created as a download \[we don’t need to download anything more than once\]\), ‘Frozen’ is ‘1’ \(ie: all associated job\_ids are frozen\), and Release\_date is &lt; now \(ie: Release date has past\).
2. For each of these SET\_IDs, first check that none of the JOB\_IDs are still under embargo, then query all DEP\_\* tables, for data with JOB\_IDs which map to this SET\_ID.
3. Export each SET\_ID set as a tarball named CHEMBLID\_XYZ \(ie: the ChEMBL\_ID for this freeze id\) to an external ftp site.
4. Set the ‘Released’ field to ‘1’.

Note that this ‘Release’ process is..

1.completely automatic. So, can be set to any day of the week, months in advance, and once set, can be relied upon to release the data on exactly the right day. 2.will fail if any of the jobs are still under embargo, or the set is not marked as ‘Frozen’. For this reason, the same nightly script also assesses which releases are imminent \(maybe 1-2 weeks away ?\), and if any are expected to fail. If any are, then chembl admin is emailed to warn of this imminent problem. 3.an export directly from the production database. So, the release does not rely on the data passing through a schema release process before it can be made available. 4.Does not contain any curated data, only raw, deposited data. It is very important that the frozen set does not contain any curated data, as this may change over time, meaning that the ‘frozen’ download set would need to be continually updated. Note, however, that the frozen set contains activity\_ids and \*IDx’s which allow researchers to subsequently map the frozen sets to the latest curated equivalents in the released ChEMBL schema.

Private export of the depositor set.

For the purposes of review for a publication, either by the author or the reviewers, it may be necessary to export a ‘tagged’ or ‘frozen’ data set at anytime. This export would run a similar process to the public export described above, except that the process would...

1.Not be automatic \(would require an administrator to run the request\) 2.Not be blocked by embargos of any kind, 3.Not require any reference to the FROZEN, FROZEN\_RELEASE\_DATE or FROZEN\_RELEASED fields in the DEPOSIT\_SET table, 4.Would create the file within an internal location within the EBI, from where an administrator could privately send \(email?\) the file to the depositor.

Tagged and Frozen sets can only be ‘owned’ by a single depositor.

As mentioned above, it is proposed that a business rule should be: “Tagged and frozen sets must never contain data from more than one depositor”. In other words, the frozen set is effectively ‘owned’ by a only a single depositor, as only they have contributed data to it.

Thus, if a depositor screened a set of GSK malaria cpds deposited by GSK, and wished to freeze the data set, then they could only freeze the activities and assays they had created. So, when referring to the data set in a paper, the authors could not simply refer to the data set as a single bundled ChEMBL ID, but must instead say something like:

“We used the set of GSK malaria cpds tagged within ChEMBL as CHEMBLID123456, and generated a frozen set of activities and assays called CHEMBL654321”.

\[NB: This example supposes that GSK elected to not ‘Freeze’ their set, but simply to ‘Tag’ it so that they may edit structures at a later date\]. Thinking ahead when depositing data.

An important requirement for the tagging/freezing functionality is that job\_ids are allowed to exist within multiple depositor sets. The reasons for this are clearer by example.

Suppose a depositor wished to create two frozen set ids, one for each of two separate publications. Suppose that they first deposited a set of compounds in job X, then subsequently deposited some assays and activities against these same cpds in job Y \(for publication 1\). Then, later, they deposited another set of assays and activities against the same cpds again in job Z \(for publication 2\). With these data in ChEMBL, they would then be able to create two frozen sets - one for each of the two different publications: The set for publication 1 would contain jobs X and Y, and the set for publication 2 would contain X and Z.

However, a complication exists if job X above, in addition to the compound set required, also contained sets of activities and assays unrelated to publication 1 or 2. In this case, these unrelated activities and assays could not be separated from the compounds of interest, and so would be bundled together with the relevant assays and activities in publications 1 and 2.

Also, suppose that for publication 3, the depositor/author used only a handful of the compounds in job X, and some new activity data \(job Q\) using a single assay from job Y. Under these circumstances it might be better to create a frozen set from just job Q, but explain that cpds and assays cited in the deposition set are available in other frozen sets \(X and Y\). The alternative to this, is to include jobs Q, X and Y altogether in one new frozen set \(but this would include many irrelevant compounds and assays and activities\).

Clearly, none of these more complicated scenarios are very satisfactory. This is an unfortunate weakness of this relatively simple, job-based model \(only a far more complex model would allow more bespoke frozen sets and embargo functionality\). To mitigate against these problems, administrators and depositors need to think ahead, and attempt to separately deposit data sets which, in future, they suspect they may wish to use in multiple frozen sets \(eg: Typically, this might mean that a batch of compounds should be usually deposited separately\). However, there is of course a limit to how far this should be taken: only by depositing every single \*IDx and activity in separate single job\_ids can complete flexibility be ensured, but this would be completely impractical to manage.

Splitting jobs: A functionality which could be added, although not advertised or encouraged might be the ability to ‘split’ a job\_id. Administrators could use this under circumstances where depositors found existing job\_id data sets difficult to work with. Splitting would not break the data model.

Other required functions for the loader to manage frozen sets

The simple data model for managing depositor set \(with two tables and a few simple constraints\), can leave open the possibility of manually editing tables in a way which has unforeseen consequences. For example, there is no database constraint that would prevent a ChEMBL admin from manually inserting a job\_id to a tagged/frozen set which was owned by another depositor.

For this reason, it is recommended that the tables are protected from manual intervention by only allowing read access to the ‘ChEMBL-administrator’ Oracle role \(but full access to the ‘Loader’ role\). If this occurs, then one must ensure that all the operations that might be required are available as methods via the loader code, under the ‘depositor’ role. Here are some of those operations:

•Ability to insert and delete job\_ids in a frozen set that has not yet been released \(using only job\_ids that belong to the owner of the frozen set\). •Option to force create new downloads \(override ‘Release’ = ‘1’\), if ftp files are lost etc •More ? Conclusion. Implementation.

Implementing the locking utility would be relatively simple, and would not require the embargo utility to be activated \(although, clearly, all data would be considered public all the time\).

For implementing the embargo utility, user ‘Roles’ for querying would be defined in a table ROLES, editable only by ChEMBL administrators, which gave mapping of role\_ids to src\_ids accessible. Single role\_ids could map to multiple src\_ids. A separate USERS \(email addresses\) table would map individual users to a role\_id. Thus a user could be granted access to any combination of src\_ids \(but usually only one\).

Enabling the embargo features would probably require substantial changes to the front end web app, perhaps even a complete re-write. \[NB: authentication etc looks v nice and comes out of the box in django!! Cx\_Oracle to set a context variable in Oracle, to set the role?\]. However, the functionality could be created and tested and be present in ChEMBL, but just not used until the front end was updated.

