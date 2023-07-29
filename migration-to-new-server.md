Migrating Existing REDCap Instance to New Server
=====================

* REDCapCon Sept 2023, Seatle WA
* Will Beasley, Thomas Wilson, Caxton Muchano, University of Oklahoma Health Science Center
* Greg Neils, MGB

This presentation is a summary of the detailed instructions at
<https://github.com/OuhscBbmc/redcap-migration/blob/main/sources/redcap-installation-public-oklahoma.md>.

Background
-------------------

* This summarizes a REDCap upgrade from an old Windows instance to a new RHEL instance.

  * Our instance was stuck on an old version of REDCap (say, v11)
    because the WAMP stack was also old and
    didn't support the newest requirements and didn't easily upgrade.
  * We started with the newest stack possible (eg, RHEL, PHP, & MariaDB versions)
    but initially installed only v11 of REDCap (the exact same as the as the old instance).
  * After a day or two of stability with live users on v11 on the new instance,
    we upgraded to the newest REDCap (say v12).

* Our scenario might be coming be more commonplace in the next few years
  at institutions with light IT support.
  (At OU, a research group is responsible for REDCap,
  and IT helps with the stack underneath.)

* !! Please think about the security & authentication involved and the vulnerable transitions between
  (a) your existing server,
  (b) the stub database on the new instance, and
  (c) the migrated database on the new instance.

* All Universities have different environments.
  Please review your migration plan with your campus's security team beforehand.
  For instance,
  it would be very bad to migrate a database full of PHI to a web server that hadn't been secured first.

* We'll cover migrating from a local server to another local server.
  See Greg & MGB's later presentation on unique challenges of migrating to Azure.

* Suggestions are welcome.
  Consider if there are better ways to accomplish these goals,
  or clearer ways to communicate the ideas.

Today's definition of "migration"
---------

* database & edocs files are moved.
* Everything else is brand new, including:
  * database server & maybe even the database engine
  * REDCap's PHP files
  * config files (eg, php.ini & apache.config)
  * authentication system possibly
    * which means usernames in database might change in ~8 tables
    * See Greg's Azure migration presentation
  * url of the server, possibly

Themes & Takeaways
-------------

* Practice many times on a test/dev instance
* Document every single command.
  * It's tedious, but very helpful when you're running through it a few times
    and forget if you did something for the current practice run, or the previous practice run.
* Our scripts are a starting point, but you'll need your own document
* Every institution's setup is different, including:
  * network topology
  * authentication
  * REDCap version
  * hardware
  * OS.
    Linux may be easier than Windows if you're more comfortable with bash than PowerShell.
    Or vice versa.
    If you're Windows using the GUI (not PowerShell), take screenshots of your steps when you practice.
* Upgrade vs Migrate:
  * Don't upgrade & migrate in the same step (eg, from v13.1.0 on local to v14.1.0 in the cloud)
  * Ideally upgrade on your old instance before migrating
  * If you migrate before you upgrade, consider staying on the old version for a few weeks before you upgrade.  It helps identify location of problems
