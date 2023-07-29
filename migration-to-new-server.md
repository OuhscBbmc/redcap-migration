Migrating Existing REDCap Instance to New Server
=====================

* REDCapCon Sept 2023, Seatle WA
* Will Beasley, Thomas Wilson, Caxton Muchano, University of Oklahoma Health Science Center
* Greg Neils, MBG

This presentation is a summary of the detailed instructions at
<https://github.com/OuhscBbmc/redcap-migration/blob/main/sources/redcap-installation-public-oklahoma.md>.

Background
-------------------

* This summarizes a REDCap upgrade from an old Windows instance to a new RHEL instance.
  Our instance was stuck on an old version of REDCap (say, v11)
  because the WAMP stack was also old and didn't support the newest requirements and didn't easily upgrade.
  We started with the newest stack possible (eg, RHEL, PHP, & MariaDB versions)
  but initially installed only v11 of REDCap (the exact same as the as the Windows instance).
  After a day or two of stability with live users on v11 on the new instance, we upgraded to the newest REDCap (say v12).

* !! Please think about the security & authentication involved and the vulnerable transitions between
  (a) your existing server,
  (b) the stub database on the new instance, and
  (c) the migrated database on the new instance.

* All Universities have different environments.
  Please review your migration plan with your campus's security team beforehand.
  For instance,
  it would be very bad to migrate a database full of PHI to a web server that hadn't been secured first.

* The new instance assumed the same public url
  (controlled by the university's load balancer),
  so we didn't have to worry about redirecting between servers.

* Suggestions are welcome.
  Consider if there are better ways to accomplish these goals,
  or clearer ways to communicate the ideas.