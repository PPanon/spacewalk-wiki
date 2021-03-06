
![Alt](images/spacewalk_25.png?raw=True)
# __Spacewalk 2.5 Release Notes__



Hello everyone,

We are proudly announcing release of Spacewalk 2.5, a systems management solution.

The download locations are

  * http://yum.spacewalkproject.org/2.5/RHEL/6/
  * http://yum.spacewalkproject.org/2.5/RHEL/7/
  * http://yum.spacewalkproject.org/2.5/Fedora/22/
  * http://yum.spacewalkproject.org/2.5/Fedora/23/

with client repositories under

  * http://yum.spacewalkproject.org/2.5-client/RHEL/5/
  * http://yum.spacewalkproject.org/2.5-client/RHEL/6/
  * http://yum.spacewalkproject.org/2.5-client/RHEL/7/
  * http://yum.spacewalkproject.org/2.5-client/Fedora/22/
  * http://yum.spacewalkproject.org/2.5-client/Fedora/23/


SUSE Linux client packages can be found here

 * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.5/openSUSE_13.2/
 * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.5/openSUSE_Leap_42.1/
 * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.5/openSUSE_Tumbleweed/
 * http://download.opensuse.org/repositories/systemsmanagement:/spacewalk:/2.5/SLE_12_SP1/

For fresh installations, please use steps from

  * https://fedorahosted.org/spacewalk/wiki/HowToInstall

If you plan to upgrade from older release, search no more -- the following page will guide you:

  * http://fedorahosted.org/spacewalk/wiki/HowToUpgrade
## Features & Enhancements in Spacewalk 2.5



  * Spacewalk now supported on Fedora 23
  * Spacewalk supports Fedora 23 clients
  * System entitlements and Software Channels entitlements were removed
  * Improved first organization creation
  * OSAD now works in failover mode via proxy
  * Plenty of small enhancements and fixes
    * 'Select All' button now correctly selects only filtered systems/packages/errata etc.
    * RDO Openstack guests are now correctly recognized as virtual guests
    * And many, many more ...
  * New API calls:
    * packages.listSourcePackages
    * packages.removeSourcePackage
    * system.scheduleLabelScriptRun
    * system.schedulePackageInstallByNevra
    * system.schedulePackageRemoveByNevra


The up-to-date API documentation can be found at http://www.spacewalkproject.org/documentation/api/2.5/
## Contributors



Our thanks go to the community members who contributed to this release:

 * Anastasios Papaioannou
 * Aron Parsons
 * Avi Miller
 * Bernhard Lichtinger
 * Dario Leidi
 * Duncan Mac-Vicar P
 * Frantisek Kobzik
 * Johannes Renner
 * Kevin Walter
 * Klaas Demter
 * Mark Huth
 * Matei Albu
 * Matej Kollar
 * Matthias Thubauville
 * Michael Brookhuis
 * Michael Calmer
 * Michael Mraka
 * Michele Bologna
 * Shannon Hughes
 * Silvio Moioli
 * Thomas Wouters

https://fedorahosted.org/spacewalk/wiki/ContributorList
## Some statistics



In Spacewalk 2.5, we've seen

    * 75 bugs fixed
    * 1071 changesets committed
    * 1408 commits done

Github repo for commits since Spacewalk 2.4

    * [Spacewalk 2.4 to 2.5](https://github.com/spacewalkproject/spacewalk/graphs/contributors?from=2015-09-29&to=2016-05-26&type=c)
## User community, reporting issues



To reach the user community with questions and ideas, please use
mailing list spacewalk-list@redhat.com. On this list, you can of
course also discuss issues you might find when installing or using
Spacewalk, but please do not be surprised if we ask you to file a bug
at https://bugzilla.redhat.com/enter_bug.cgi?product=Spacewalk with more
details or full logs.

Thank you for using Spacewalk.