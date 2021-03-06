# Delta RPMS

## Tasks (in order)





|  Task  |  Estimated Hours  |  Assignee  |  Status  |  Notes  |
| --- | --- | --- | --- | --- | --- |
|  Modifying repomd generation to be performed as asynchronously, not on demand  |  25  |    |   |   |  |
|  Presto MD generation (including config option to disable)  |  25  |     |   |    |  |
|  Delta RPM Generation  |  25  |     |   |    |  |
|  Clearing old drpms  |  5  |   |   |   |  |
|  Client side testing/development   |  5  |   |   |   |  |
## Requirements

 1.  If a system has the presto yum plugin installed and is configured to get updates from the spacewalk server, any package update request should be  fulfilled with a delta rpm.

 2.  There shall be a way to disable delta rpms from being offered from the server.  
 3.  The system shall automatically clear old delta rpms.
## Specification
 


 * Backend 
  * Currently the yum repo data is taking too long to generate.  It isn't created until a client requests it and the amount of time to create it is greater than the yum timeout.  We need to move this to a background daemon such that clients can still get updates (albeit old) until the process is complete, at which point they will have access to the new repomd file.
  * Generate the presto repo data.  This will be done in the same process as the aforementioned yum repo metadata.  Note, this step requires the drpm md5sum.
  * Generate the drpms.  We cannot generate these on the fly because the presto repodata requires the md5sum of the drpm.  This will need to be done in the background daemon process as well before the presto repomd is generated.  These will be saved in /var/cache/rhn/drpms/N(0..2)/NAME/md5sum-md5sum.drpm.
  * Provide a config option to disable/enable presto rpm support on the server side.
  * This background daemon called (spacewalk-repomd) will be started with spacewalk and will check every (5 minutes?) to see if repodata needs to be regenerated.  If it does, it will do the following (in somewhat this order):
   1. Generate yum repomd
   2. Generate drpms
   3. Generate presto repomd
   4. Clear old orphan drpms



 * Client side
  * This most likely "just works".  Once we have the backend stuff to test, we can actually do some testing.  
## FAQ

* What is a delta rpm?

   * It is a collection of the binary diffs between all the individual files of two different versions of the same package with the rpm header of the new package prepended to the file.

* Can rpm work directly with drpms?
   * No, the full rpm must be recreated before installing.  Rpm itself has no idea what a drpm actually is and will try (and fail) to install it like a normal rpm.

* Can yum work directly with drpms themselves?
   * No, yum (with the presto yum plugin or a similiar plugin) can only work with drpms if they are in an appropriate repo.  The plugin itself recomposes the rpm into its original form before having yum install it.  
## Common Commands



Needed Packages: presto-utils, deltarpm, yum-presto

    Create a drpm:
         makedeltarpm  oldrpm.rpm  newrpm.rpm  deltarpm.drpm 

    Create drpms from a full repo:
    	createdeltarpms  ./repo ./repo/DRPMS 

    Reconstruct original rpm using on disk installation:
     	applydeltarpm  delta.drpm  new.rpm 

    Reconstruct original rpm using an old rpm:
     	applydeltarpm  -r old.rpm  delta.drpm  new.rpm 

    Create a presto repo (and the prestodelta.xml file):
    	createprestorepo ./repo 

    Then to finish the deal and link the prestodelta.xml to the repomod.xml file:
             modifyrepo  ./repodata/prestodelta.xml  ./repodata/ 
## My thoughts



 * I like the idea of delta rpms 'just being there' (i.e. so the user wouldn't have to do anything to get them there, or at least wouldn't have to generate them themselves and upload them to the sat).  
 * Since it does require extra cpu and memory resources to rebuild the rpm on each client, it should be an optional on the satellite to provide/build/publish deltarpms.


