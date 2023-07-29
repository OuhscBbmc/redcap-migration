Migrating a REDCap Instance
=================

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

* Several sections have been simplified or removed for clarity.
  For instance we omitted the description of our practice cycles with the dev server.

* Suggestions are welcome.
  Consider if there are better ways to accomplish these goals,
  or clearer ways to communicate the ideas.

Stack before REDCap
-------------------

* Create VMs (pretend our internal domain is "ounet.edu")

  * prod-2-db.ounet.edu
  * prod-2-web.ounet.edu
  * token-guide-1.ounet.edu (optional -- if you want a token server)

* Install Apache (on the web server)

  * Check Apache is listening from shell: `netstat -veepantu | grep 80`

* Install PHP (on the web server)

* Install MariaDB (on the db server)

  * The documentation [describes several approaches](https://mariadb.com/kb/en/getting-installing-and-upgrading-mariadb/) for each common environment
  * We used [dnf on RHEL](https://mariadb.com/kb/en/yum/)

* Edocs storage

  * The Edocs directory contains a bunch of user-uploaded files and is commonly 20+ GB.  The "File Upload Settings" page in REDCap's Control Center describes various options.  Here's one approach.
  * Created a a networked file share (say, "//fileshare/").
  * On the new web server, mount a file share from "//fileshare/rc-storage/prod-2"  to "/var/www/rc-storage/prod-2/" so it looks local to the OS.
  * Talk to your security team about the best location for your environment.  Make sure it's not publicly visible on the web server (say in "/var/www/html/*").
  * In the Windows web server (that will be discontinued), map the *R* drive to `\\fileshare\rc-storage\`

* Requests to Campus IT for Networking

  * firewall exception for `prod-2-web` to `prod-2-db`
  * firewall exception for `prod-2-web` to  REDCap's Community site (to download upgrades)
  * firewall exception for `prod-2-web` to the Edocs location.
  * firewall exception for `token-guide-1` to `prod-2-db` (optional -- for token server below)
  * load balancer   -- but wait until both servers are secured for PHI!

* Strategy for transferring files

  * You may need to move some new files to both RHEL servers, such as the (a) REDCap installation files to the web server, (b) SQL scripts to install & upgrade the database, and (c) configuration files.
  * It is a small security risk to allow the servers to access a bunch of external servers.  It's also tedious to submit a ticket for each new firewall exception.
  * Instead, use your local desktop as the middleman.  Download the files to your desktop first, and then transfer them to the server with a protocol like [scp](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/).  If the local machine is Windows, a program like [WinSCP](https://winscp.net/eng/index.php) makes this an easy drag & drop.

* Install GNOME (optional)

  * Institutions staffed with enough Linux experts will not need a desktop environment.
    But if the REDCap admins are coming from Windows,
    something like [GNOME](https://www.gnome.org/) may be desirable.
  * Installing a desktop environment increases the vulnerability surface and therefore theoretically increases risk.
    (Like installing almost any additional software.)
    But that risk is probably minimal.
  * On the other hand, those uncomfortable with command line administration will be more productive initially
    because the visual metaphors will be familiar to them.
    Furthermore, a desktop environment might *decrease* the practical risk among new Linux admins,
    because they'll be less likely to make mistakes
    (eg, moving a sensitive file into the wrong directory).

* Install DBeaver (assuming on some RHEL VM)

  * Alternatives are [phpMyAdmin](https://www.phpmyadmin.net/) and the basic
    [MySQL Command Line Client](https://dev.mysql.com/doc/refman/8.0/en/mysql.html).
    It won't be used much after the server is deployed unless you're doing a lot of unconventional development.
  * Install the Java prereq: `sudo dnf install java-11-openjdk`
  * Install DBeaver: `sudo rpm -i dbeaver-ce-*-stable.x86_64.rpm`
  * Install JDBC driver
    * [Download](https://mariadb.com/kb/en/about-mariadb-connector-j/) the latest MariaDB J connector on a local machine
    * Transfer to db server with scp.
    * Within DBeaver's "Driver settings" connection dialog box, point to the new jar.
  * Depending on the host VM, a firewall exception might be necessary.

Install REDCap and Establish Stub Database
----------------------------

* Take a VM snapshot (on the web and db server) before starting REDCap installation.

* Download installation from <https://redcap.vanderbilt.edu/community/custom/download.php> to `~/Downloads/redcap12.0.0.zip`

* Extract/unzip to `~/Downloads/redcap/`

* Move directory to `/var/www/html/` (working directory is "~/Downloads/")

  ```sh
  sudo cp -r redcap/ /var/www/html/
  ```

* From a client's browser, point to <http://prod-2-web.ounet.edu/redcap/install.php>
  which will walk you through the installation.

* Create blank database

  * Use scp to transfer the ~10 lines of DDL SQL from the browser to the db server and execute.
  * Modify the script if desired.  For example, limiting redcap_user's access to only "prod-1.ounet.edu"
  * Set values in `/var/www/html/redcap/database.php`, as described in <http://prod-2-web.ounet.edu/redcap/install.php>.

* Establish empty database tables

  * Return to <http://prod-2-web.ounet.edu/redcap/install.php> or <http://rc-dev-1-web.ounet.edu/redcap/install.php>
  * Use scp to transfer the ~5k lines of DDL SQL (from install.php) to a sql file on the db server and execute.

* Connect to LDAP.

  * In the User Security Config page, change to "LDAP + Table Based"
  * In the previous prod server, go to the LDAP troubleshooting page to get the info on its REDCap service account.
    (Sorry I don't know how to do this for fresh installs without an existing file.
    The REDCap config page provides links and help.)
  * Use the information in `/var/www/html/redcap/webtools2/ldap/ldap_config.php`
  * It's important to copy selective content into the new file.
    Do NOT copy & replace the whole file because the LDAP versions may have changed in the meantime.
  * Restart the server: `sudo systemctl restart httpd`

* Swap Admin Accounts

  * This may be an unconventional approach.
    Once LDAP+Table is activated, the "site_admin" isn't the default login through the browser,
    so your LDAP-backed admin accounts aren't initially allowed to log in.
  * In the "Security & Authentication" page, change to "LDAP + Table Based".
  * Activate an elevated/admin account using the REDCap login screen.  These accounts won't have super user privileges yet.
  * In the database table `redcap_user_information`, change `super_user` and related columns from 0 to 1.
  * In the "Administrator Privileges" page, remove all privileges from "site_admin".
  * The Community page
    [How to change and set up authentication in REDCap](https://redcap.vanderbilt.edu/community/post.php?id=691)
    describes *How to change from "None (Public)" to "LDAP" authentication*,
    but it doesn't seem to address the catch-22 we encountered (as of July 2022).
    Maybe I'm missing something.

* Check REDCap's "Configuration Check" page (<http://prod-2-web.ounet.edu/redcap/redcap_v[[version]]/ControlCenter/check.php>)

* Point to Edocs folder

  * In the REDCap browser, go to "System Configuration" | "File Upload Settings"
  * In "File Storage Location" dropdown, keep it at "Local (on REDCap web server)
  * In "Local Server File Storage": `/var/www/rc-storage/prod-2/edocs/`.  The trailing slash is important.

* Grant write access to the web root directory.
  Otherwise REDCap's config check produces the errors
  *"temp" directory is NOT writable - CRITICAL* &
  *"modules" directory is NOT writable - CRITICAL*.
  We run Apache through the "systemd" service manager,
  which is the recommended approach for RHEL.

  ```sh
  sudo chown apache:apache -R /var/www/html
  sudo systemctl restart httpd
  ```

  Other discussions on the Community site:

  * How do I set temp folder for RedCap in Linux?:
  <https://redcap.vanderbilt.edu/community/post.php?id=21782>
  * Modules not writable- critical message:
  <https://redcap.vanderbilt.edu/community/post.php?id=126750>
  * edocs,modules and temp folders show directory is NOT writable - CRITICAL message:
  <https://redcap.vanderbilt.edu/community/post.php?id=87321>
  * "Web server's temp directory is NOT writable" Configuration message:
  <https://redcap.vanderbilt.edu/community/post.php?id=3838>

* Tweak PHP configuration on web server

  * Find location of [PHP.ini](https://stackoverflow.com/a/8684638/1082435): `php --ini`
  * Change `max_input_vars` to "100000".  Make sure to remove the leading semicolon to uncomment.  (Located around line 404.)
  * Change `post_max_size` to "32M".  (Located around line 694.)
  * Change `upload_max_filesize` to "32M". (Located around line 846.)
  * Change `;session.cookie_secure` to `session.cookie_secure = On`.  Remove the leading semicolon to uncomment.
  * Change location settings for your site.
    (Located around line 240.)
    Remove the leading semicolon to uncomment including
    `date.timezone` (ie, "America/Chicago"),
    `date.default_latitude`, &
    `date.default_longitude`.
  * Restart Apache & PHP service: `sudo systemctl restart httpd php-fpm`
  * (Restarting httpd by itself didn't update max_input_vars, so we rebooted the whole OS.)
  * Some of these are hard to find and even are duplicated.
    Make sure you get them all.
    To search for "post_max_size", run this (assuming you have the same location of php.ini).

    ```sh
    /etc/php.ini | grep -n post_max_size
    ```

* Tweak MariaDB configuration on database server

  * Find location of [my.cnf](https://stackoverflow.com/questions/2482234/how-do-i-find-the-mysql-my-cnf-location):
    `mysqladmin --help`
  * Change `query_cache_limit` to 16777216.  Create the entry if it doesn't exist.
  * Restart MariaDB: `sudo systemctl restart mariadb.service`

* Add cronjob using syntax from REDCap's suggestions.  Click the "Go to Cron Jobs page" link on the configuration check:

  ```sh
  sudo crontab -l
  sudo crontab -e
  ```

* Temporarily turn on diagnostics & development logging on the web server, if necessary.
  Make sure to turn it off before real production use,
  which is described in the "Immediately Before Go Live" section near the end of this document.

  * `/var/www/html/redcap/redcap_connect.php` line 5: "error_reporting(0)" to "error_reporting(1)".

  * Create a <phpinfo.php> file.
  [Maybe in the future](https://redcap.vanderbilt.edu/community/post.php?id=98722)
  it won't be necessary.

    * Create a text file `~/Downloads/phpinfo.php` with
    [two lines of code](https://redcap.vanderbilt.edu/community/post.php?id=98695):

      ```php
      <?php
      phpinfo();
      ```

    * Copy

      ```sh
      sudo cp ~/Downloads/phpinfo /var/www/html/redcap/phpinfo.php
      ```

  * Turn on PHP logging in `/etc/php.ini`

    * Change `display_errors = Off` to `display_errors = On` (be careful it's in the file twice)
    * Change `display_startup_errors = Off` to `display_startup_errors = On` (be careful it's in the file twice)

  * View the running tail end of the error or access logs for Apache
    (not necessarily for PHP, unless PHP is passing them on to Apache)

    ```sh
    sudo tail -f /var/log/httpd/error_log
    # or
    sudo tail -f /var/log/httpd/access_log
    ```

  * View the Apache status: `sudo systemctl status httpd`

  * Disable [libvirtd](https://libvirt.org/manpages/libvirtd.html), which was installed by default.

Immediately Before Migration
------------

* Notify Users (ideally with 1+ days of warning)

* Change to 'System Offline' in the REDCap Control Center's General Configuration.

Migrate Database & Edocs
------------

* !! Security Considerations

  * This section introduces PHI on the new server and therefore raises the risks considerably.
    Make sure the new server is completely secured.  It's safer if the web server isn't publicly available yet.

  * As soon as `/www/html/redcap/database.php` points to the migrated database below,
    the authentication mode switches to the old instance's settings.
    If those settings aren't secure in your new environment, PHI is at risk.

  * You might need to modify the settings on the old instance before you migrate.
    Alternatively you might need to
    (a) upload the database,
    (b) change values in the `redcap_config` table (using, DBeaver, phpMyAdmin, or the mysql command line),
    and finally
    (c) point database.php to the migrated database.

  * Review this section with your institution's security team.

* Take a VM snapshot.

* So far, we've been using a stub of a database.
  This stage moves the existing database from the previous server and plops it down in the new one,
  and then points REDCap's PHP to the new-old database.

* Copy database from prod's predecessor

  * Use a wizard in MySQL Workbench to create this command,
    which saves as 20+ GB plain-text sql file to "R:\prod-2".
    The cnf referenced below was a simple file containing the password.

    ```sql
    mysqldump.exe --defaults-file="c:\users\...\tmp2weifd.cnf"  --user=redcap_user --max_allowed_packet=1G --host=127.0.0.1 --port=3306 --default-character-set=utf8 "redcapv1" > R:\prod-2\redcapv1.sql
    ```

    ![export](images/db-export-workbench.png)

  * Alternatively, use PowerShell.

    ```ps
    > mysqldump -u redcap_user -p --opt redcapv1 > R:\prod-2\redcapv1.sql
    ```

  * If you want to look for the
    [location of the data on the server](https://www.tutorialspoint.com/how-to-find-the-mysql-data-directory-from-command-line-in-windows),
    execute this sql: `SELECT @@datadir`

* Possibly modify the sql file before importing.  Either of these modifications might necessary.

  * **Changing Encoding**: The
    [file encoding might need to be converted to ASCII](https://stackoverflow.com/a/41264890/1082435).
    It happened in one of our practice runs and we could tell what parameter triggered it.
    There were a few possibilities, and some aren't supported on older MySQL versions.

    ```sh
    # didn't work:
    # iconv -f utf-16 -t utf-8 ~/Downloads/redcapv1-utf16.sql > ~/Downloads/redcapv1-utf8.sql
    # did work
    iconv -f utf-16 -t utf-8 ~/Downloads/redcapv1.sql > ~/Downloads/redcapv1-utf8.sql
    ```

  * **Changing Database Name**:
    *If* the dump file contains the source database name and the destination database is different,
    update the database name in the text file.
    The file can be huge and not feasibly loaded in a text editor like Notepad++ to find & replace the text.

    Use a command line utility like [sed](https://www.gnu.org/software/sed/manual/sed.html)
    to uses a regex to substitute the new database name for the old one.
    We didn't remove the name completely, because it appeared in different forms like "\`redcapv1\`." and just "redcapv1".

    In this case, the term `redcapv1` from the "redcapv1.sql" file is replaced by the term
    `redcapv2` and written to the "redcapv2.sql" file.

    ```sh
    sed 's/redcapv1/redcapv2/g' < redcapv1.sql > redcapv2.sql
    ```

    This will take a few minutes if the database is several GB.
    Loosely monitor the progress in another shell by comparing the two file sizes with `ls -al`

* Import the database to the existing (and empty) "redcapv2" database

  ```sh
  mysql -u root -p redcapv2 < ~/Downloads/redcapv1-try1-utf-8.sql
  ```

  This will take a few hours if the database is several GB.  Loosely monitor the...

  * disk IO in another shell with [iotop](https://www.tecmint.com/iotop-monitor-linux-disk-io-activity-per-process/)
    which came installed with our RHEL.
    See *instant* activity with `sudo iotop -o` or *accumulated* activity with `sudo iotop -oa`.

  * [sizes of the databases](https://stackoverflow.com/a/1733523/1082435) with sql:

  ```sql
  SELECT
    table_schema                                            as db_name,
    round(sum(data_length + index_length) / 1024 / 1024, 1) as db_size_mb
  FROM information_schema.tables
  GROUP BY table_schema;
  ```

* Grant privileges to "redcap_user" to the migrated database.
  This sql command mimics the last line of the "Create New Database" bullet above (copied from install.php).

  ```sql
  GRANT select, insert, update, delete on `redcapv2`.* TO 'redcap_user'@'%';
  ```

* In the migrated database's `redcap_config` table:

  * Change `redcap_base_url` to the new name (eg, "http://prod-2-web.ounet.edu/redcap")

  * Update `edoc_path`.

  * Code:

    ```sql
    use redcapv2
    UPDATE redcap_config SET value = 'http://prod-2-web.ounet.edu/redcap/' WHERE field_name = 'redcap_base_url';
    -- UPDATE redcap_config SET value = 'https://bbmc.ouhsc.edu/redcap/' WHERE field_name = 'redcap_base_url'; -- Through the load balancer.
    UPDATE redcap_config SET value = '/var/www/rc-storage/prod-2/' WHERE field_name = 'edoc_path';

    SELECT field_name, value FROM redcap_config
    WHERE field_name in ('redcap_base_url', 'edoc_path')
    ```

* In `/www/html/redcap/database.php`, change the value of "db" to "redcapv2".

* To facilitate REDCap's Easy Upgrade feature, follow the config page's directions for creating a new user.
  Note that you may need to add additional databases for the user if you do more migrations.

* Copy edocs from prod's predecessor to "//fileshare/rc-storage/prod-2/edocs"
  (or "R:\prod-2\edocs" if you mapped the drive as described above).

Token Server
------------

* Install and configure the
  MariaDB ODBC [driver](https://mariadb.com/kb/en/mariadb-connector-odbc/)
  on the client server, "token-guide-1.ounet.edu".
  Currently on the [download page](https://mariadb.com/downloads/connectors/connectors-data-access/odbc-connector)
  the 'Product' is called "ODBC connector".

* Make sure firewall exceptions have been granted. (Probably by filing an IT request ticket.)

* Create a dedicated user account that has access from one client (called "token-guide-1.ounet.net").
  Replace `password` with the real value.

  ```sql
  use redcapv2
  CREATE USER 'token'@'token-guide-1.ounet.edu' IDENTIFIED BY 'password';
  GRANT ALL ON redcap_user_rights to 'token'@'token-guide-1.ounet.edu' IDENTIFIED BY 'password' WITH GRANT OPTION;

  FLUSH PRIVILEGES;
  ```

  * The token-guide machine uploads tokens to the token server,
    further described in the [Security Database](https://ouhscbbmc.github.io/REDCapR/articles/SecurityDatabase.html) vignette.

Immediately Before Go Live
------------

* Do these *before* opening up the firewall exception.
  Avoid having these enabled on an accessible web server because they could disclose sensitive information.

* Turn off diagnostics & development logging

  * `/var/www/html/redcap/redcap_connect.php` line 5: "error_reporting(1)" to "error_reporting(0)".

  * `sudo rm /var/www/html/redcap/phpinfo.php` (or any other place you may have it)

  * Turn off PHP logging in  `/etc/php.ini`

    * Change `display_errors = On` to `display_errors = Off`
    * Change `display_startup_errors = On` to `display_startup_errors = Off`
    * Possibly change `error_log = php_errors.log` to `;error_log = php_errors.log`

* Establish regular automated backups (at the level of the database and/or VM)

Go Live on 11.0.0
------------

* Connect to university load balancer and allow access from the outside world

* Change to 'System Online' in the REDCap Control Center's General Configuration.

* Notify Users

Upgrade to 12.0.0
------------

* After a day or two of stable behavior, upgrade to the newest version of REDCap (say, 12.0.0)

* Assuming the older version of REDCap doesn't have a major security vulnerability,
  we recommend splitting these two hops on different days
  (first leg: from old to new hardware; second leg: from old to new REDCap version).
  This will help isolate & identify the source of any migration problems.

Maintenance
-----

* Establish and test backup routines --at the level of the VM and/or database.

* Perform & review periodical vulnerability scan reports to identify available security patches
  and updates for Linux and its packages.
  Especially those involved with REDCap's public-facing web server duties.

* Every week, review the REDCap's [ChangeLog](https://redcap.vanderbilt.edu/community/post.php?id=13)
  and decide if your institution should upgrade.
  Pay particular attention to the (occasional) entries labeled as "Major bug fix" or "Security fix".

Helpful [Community](https://redcap.vanderbilt.edu/community/) Resources
------------

* [migrating to new server process](https://community.projectredcap.org/topics/7077/migrating%20to%20new%20server%20process.html) tag

* [edocs directory](https://community.projectredcap.org/topics/14951/edocs-directory.html) tag

* [Migrating Redcap to a new server](https://redcap.vanderbilt.edu/community/post.php?id=7078):
  Sean Banks provides in a good 18-step overview that strongly influenced this plan;
  Rob Taylor and other discuss scenarios where the new instance will have a different url.

* [Migrate projects from 2 different versions of REDCap](https://redcap.vanderbilt.edu/community/post.php?id=80425):
  Alex describes how to revive an old server that can't make a single upgrade to REDCap & PHP.

* [migrate redcap to a new web server vis-a-vie external modules](https://redcap.vanderbilt.edu/community/post.php?id=82061):
  David Rodgers & Mark McEver briefly discuss migrating external modules between servers.
