Migration
=================

30 min total

* overview of migration ~5 min
* local to local migration ~5
* local to cloud migration preview ~2
  * essentially a plug for [REDCap & Azure](https://redcap.vanderbilt.edu/surveys/?__report=RPK8CL8NETYCXEYK) presentation
* themes overall lessons ~10
* questions ~5

Definition of "migration"
---------

* database & edocs files are moved.
* Everything is brand new
  * database server & maybe even the database engine
  * REDCap's PHP files
  * config files (eg, php.ini & apache.config)
  * authentication system, possibly (which means usernames in database might change in ~8 tables)
  * url of the server, possibly

Local to Cloud
-------------

Additional concerns (beyond local to local)

* Network security when transferring files

Themes
-------------

* Practice many times on a test/dev instance
* Document every since command.
  * It's tedious, but very helpful when you're running through it a few times and forget if you did something for the current practice run, or the previous practice run.
* Every institution's setup is different.
  * Our scripts are a starting point, but you'll need your own document
  * network topology
  * authentication
  * redcap versions
  * hardware
  * OS.  Linux may be easier than Windows if you're more comfortable with bash than PowerShell.  Or vice versa.  If you're Windows using the GUI (not PowerShell), take screenshots of your steps when you practice.
* Upgrade vs Migrate:
  * Don't upgrade & migrate in the same step (eg, from v13.1.0 on local to v14.1.0 in the cloud)
  * Ideally upgrade on your old instance before migrating
  * If you migrate before you upgrade, consider staying on the old version for a few weeks before you upgrade.  It helps identify location of problems

* Greg's steps
  * Establish hardware
  * Test hardware
  * Put real data into it
  * Compare old vs new platform
  * Go-live
    * take offline
    * migrate database & edocs
    * DNS update: have the url point to the new hardware's IP address
    * test & verify with real data
    * decide if you're ready to go online
      * if yes: go online
      * if not: point to old hardware's IP address (and turn it back on)

Links
---------

* [tentative schedule](https://redcap.vanderbilt.edu/surveys/?__report=RPK8CL8NETYCXEYK)
* [about REDCapCon](https://projectredcap.org/about/redcapcon/)
