Migrating Existing REDCap Instance to New Server
=====================

* REDCapCon Sept 2023, Seatle WA
* Will Beasley<sup>1</sup>,
  Thomas Wilson<sup>1</sup>,
  Greg Neils<sup>2</sup>,
  Caxton Muchano<sup>1</sup>,
  Patrick Sandin<sup>3</sup>,
  April Dickson<sup>3</sup>

* Affiliations:
  <sup>1</sup>[University of Oklahoma Health Science Center, Biomedical and Behavioral Methodology Core](https://www.ouhsc.edu/bbmc/),
  <sup>2</sup>[Mass General Brigham](https://rc.partners.org/research-apps-and-services/collect-data),
  <sup>3</sup>[University of Oklahoma IT](https://www.ou.edu/ouit).

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

* !! Please consider the security & authentication involved and the vulnerable transitions between
  (a) your existing server,
  (b) the stub database on the new instance, and
  (c) the migrated database on the new instance.

* All institutions have different environments.
  Please review your migration plan with your campus's security team beforehand.
  For instance,
  it would be very bad to migrate a database full of PHI
  that's connected to a web server that hadn't been secured first.

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

First establish stack underneath REDCap
-------------------

* Create VMs for database & web server (and optional token server)
* Install Apache (on the web server)
* Install PHP (on the web server)
* Install MariaDB (on the db server)
* Requests to Campus IT for networking
* Establish edocs storage & connections
* Strategy for transferring files
* Install GNOME (optional for Linux)
* Install DBeaver (assuming on some RHEL VM)

Requests to Campus IT for networking
------------

* firewall exception for `prod-2-web` to `prod-2-db`
* firewall exception for `prod-2-web` to REDCap's Community site (to download upgrades)
* firewall exception for `prod-2-web` to the edocs location.
* firewall exception for `token-guide-1` to `prod-2-db` (optional -- for token server below)
* load balancer   -- but wait until both servers are secured for PHI!

Strategy for transferring files
------------

* You may need to move some new files to both RHEL servers, such as the
  (a) REDCap installation files to the web server,
  (b) SQL scripts to install & upgrade the database, and
  (c) configuration files.
* It is a small security risk to allow the servers to access a bunch of external servers.
  It's also tedious to submit a ticket for each new firewall exception.
* Instead, use your local desktop as the middleman.
  Download the files to your desktop first, and then transfer them to the server with a protocol like
  [scp](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/).
  If the local machine is Windows, a program like
  [WinSCP](https://winscp.net/eng/index.php) makes this an easy drag & drop.

Install GNOME (optional for Linux)
------------

* Institutions staffed with enough Linux experts will not need a desktop environment.
  But if the REDCap admins are coming from Windows,
  something like [GNOME](https://www.gnome.org/) may be desirable.

Cons of desktop environment:

* Installing a desktop environment increases the vulnerability surface and
  therefore theoretically increases risk.
  (Like installing almost any additional software.)
  But that risk is probably minimal.

Pros of desktop environment:

* People who are uncomfortable with command line administration will be more productive initially
  because the visual metaphors will be familiar to them.
* A desktop environment might *decrease* the practical risk among new Linux admins,
  because they'll be less likely to make mistakes
  (eg, moving a sensitive file into the wrong directory).

Separate "Upgrade" step and "Migrate" step
------------

* Reason: don't change two things at once.
* Don't upgrade & migrate in the same step (eg, from v13.1.0 on local to v14.1.0 in the cloud)
* Ideally upgrade your old instance before migrating
* If you migrate before you upgrade, consider staying on the old version for a few weeks before you upgrade.  It helps identify location of problems.
* Remember that REDCap's database is updated with DDL commands on tables populated with live data.

Themes & Takeaways
-------------

* Practice many times on a test/dev instance
* Document every single command.
  * It's tedious, but very helpful when you're running through it a few times
    and forget if you did something for the current practice run, or the previous practice run.
  * It's much quicker in the long run.
* Our scripts are a starting point, but you'll need your own document
* Every institution's setup is different, including:
  * network topology
  * authentication
  * REDCap version
  * hardware
  * OS.
    Linux may be easier than Windows if you're more comfortable with bash than PowerShell.
    Or vice versa.
    If you're installing on Windows with the GUI (not with PowerShell),
    take screenshots of your steps when you practice.