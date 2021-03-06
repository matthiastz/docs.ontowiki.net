---
title: VirtuosoBackend
tags: [ontowiki, install]
sidebar: ontowiki_sidebar
permalink: /VirtuosoBackend.html
editme_path: ontowiki/VirtuosoBackend.md
---
This page explains the necessary steps to use OntoWiki with [OpenLink Virtuoso](http://virtuoso.openlinksw.com/wiki/main/Main/).
Especially, it covers the following topics:

* Install Virtuoso
* Configure OntoWiki to use Virtuoso
* Other Virtuoso backend specific details

## Overview
This is the basic procedure. For details see below.

* Check out the OntoWiki source code
* Download [Virtuoso Open-Source Edition](http://sourceforge.net/projects/virtuoso/) and compile it according to the instructions for your operating system. Make sure to get the latest stable branch available.
* Install and configure Virtuoso.
* Set up ODBC and make sure, PHP can connect to Virtuoso via ODBC.
* Set up OntoWiki to be used with Virtuoso.

## Compiling Virtuoso
For some OS, e.g. Windows, binaries exist, but might be a bit dated. If you have to compile it, it is recommended to give a `<prefix>`, which will be the path where Virtuoso will be installed. 

**Debian/Ubuntu** recommendation: 
 - For a standard install, a prefix of '/usr/local' will do just fine. This installs Virtuoso in the standard program path, and reduces confusion as to which binary to run. Also, the stock Debian or Ubuntu startup scripts can be used with [minor modifications]({{ "Custom-startup-script-for-Debian.html" | prepend:site.baseurl }}).
 - For a development environment it might be better to set things up with a named prefix folder in the '/opt' tree, i.e. `./configure --prefix=/opt/virtuoso`. This provides great flexibility for a development environment. To name but a few benefits:
  - You may uninstall the compiled software by simply removing the whole directory
  - You may test and use two different versions of Virtuoso at the same time and try out new features
  - You may choose to `chown` the whole virtuoso directory so you can run virtuoso in user-space instead of  as root
 - Virtuoso also comes along with a good README with further instructions. 
 - Please also see the notes for compiling [on Debian](http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSDebianNotes#Building from Upstream Source) and [Ubuntu](http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSUbuntuNotes#Building from Upstream Source).

**Debian/Ubuntu Tip**:  configuring with the libreadline library allows for better characters passing to isql command line client '--with-readline=path/to/libreadline' (use ` apt-get install libreadline5-dev` and '--with-readline=/usr/lib/libreadline.so').

```shell
./configure --prefix=/opt/virtuoso-opensource-version --with-readline=/usr/lib/libreadline.so
make
sudo make install
```

Nevertheless, you have to configure the DirsAllowed variable and configure ODBC. Or get the official debian package from Debian:squeeze repository. See: http://packages.debian.org/squeeze/virtuoso-opensource

Compiling Virtuoso requires the following build tools be installed:
[autoconf 2.57](http://www.gnu.org/software/autoconf/),
[automake 1.7](http://www.gnu.org/software/automake/),
[libtool 1.5](http://www.gnu.org/software/libtool/),
[flex 2.5.33](http://flex.sourceforge.net/),
[bison 2.3](http://www.gnu.org/software/bison/),
[gperf 2.7.2](http://www.gnu.org/software/gperf/),
[gawk 3.1.1](http://www.gnu.org/software/gawk/),
[m4 1.4.1](http://www.gnu.org/software/m4/),
[make 3.79.1](http://www.gnu.org/software/make/) and
[OpenSSL 0.9.7](http://www.openssl.org/)

**Debian/Ubuntu**: 
`sudo apt-get install autoconf automake libtool flex bison gperf gawk m4 make openssl libssl-dev`

Having these tools, compiling and installing Virtuoso is just a matter of configure/make/make install.

You can skip section "Install Virtuoso" and go directly to **"Configuring Virtuoso"**!

## Install Virtuoso

**Only read further if you dont read section "Compiling Virtuoso"!** There is a good installation manual on Virtuoso's page: http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSUbuntuNotes

**Ubuntu**

simple way (for Virtuoso 6.1 for now) through the packet manager

    sudo apt-get install virtuoso-opensource

## Configuring Virtuoso

In some cases you have to manually start the Virtuoso server.

In **Ubuntu** execute (Maybe you have to replace the 6.1 with the current version)
`sudo service virtuoso-opensource-6.1 start`

Virtuoso  keeps all the files for a database in a folder along with a configuration file. An example configuration file which we will use as a base for our configuration is provided with the default database. To create a Virtuoso database for OntoWiki perform the following steps:

* Create a folder named "ontowiki" under `<prefix>/var/lib/virtuoso/`.
* Copy the file `<prefix>/var/lib/virtuoso/db/virtuoso.ini` to `<prefix>/var/lib/virtuoso/ontowiki/virtuoso.ini`
* Open `<prefix>/var/lib/virtuoso/ontowiki/virtuoso.ini` in a text editor
* Find the option "DirsAllowed", it is a comma-separated list of dirs from which Virtuoso is allowed to open files. Add your PHP's temp directory (usually `/tmp`) and your OntoWiki directory under your webroot.

## Setting up ODBC
Database connection to Virtuoso happens through ODBC. ODBC is configured by two files, `odbc.ini` and `odbcinst.ini`.
On Linux systems they can be found under `/etc`, on Mac OS X systems under `/Library/ODBC`.
In case they don't exist, create them.

Add the following lines to the **odbcinst.ini** file:

    [virtuoso-odbc]
    Driver = <prefix>/lib/virtodbc.so

Add the following lines to the **odbc.ini** file:

    [ODBC Data Sources]
    VOS = Virtuoso
    
    [VOS]
    Driver = virtuoso-odbc
    Description = Virtuoso Open-Source Edition
    Address = localhost:1111

Now make sure, PHP can connect to Virtuoso via ODBC.
On some Linux systems you may have to install the ODBC package for PHP (`php5-odbc`).

Newer versions of OntoWiki come with a script to test the connectivity to Virtuoso. Just type the following on the command-line while in the OntoWiki root directory:

    make odbctest

If this gives you an error like

    make: *** No rule to make target `odbctest'.  Stop.

try the following to test the connection: Create a file called `odbctest.php` in you webroot with the following PHP code in it:

    <?php
    $conn   = odbc_connect('VOS', 'dba', 'dba');
    echo odbc_errormsg();
    $query  = 'SELECT DISTINCT ?g WHERE {GRAPH ?g {?s ?p ?o.}}';
    $result = odbc_exec($conn, 'CALL DB.DBA.SPARQL_EVAL(\'' . $query . '\', NULL, 0)');
    ?>
    <ul>
    <?php while (odbc_fetch_row($result)): ?>
        <li><?php echo odbc_result($result, 1) ?></li>
    <?php endwhile; ?>
    </ul>

Execute it by clicking on this link: [http://localhost/odbctest.php](http://localhost/odbctest.php).
You should see  a list of graphs stored in your Virtuoso RDF store.
On a vanilla Virtuoso installation this list should include the following two graphs:

* http://www.openlinksw.com/schemas/virtrdf#
* http://localhost:8890/DAV

If you see this list and no error messages along the way, go ahead configuring OntoWiki.

## Configuring OntoWiki
Perform the following steps to set up OntoWiki:

* Copy `config.ini.dist` to `config.ini` and open it in a text editor
* Under `[private]`, set the following options: 

    store.backend = virtuoso
    store.virtuoso.dsn = VOS
    store.virtuoso.username = dba
    store.virtuoso.password = dba

* Optional 1: to enable OntoWiki's debug mode, add the line `debug = yes`
* Optional 2: If you didn't change any other config option, create the following directories relative to your OntoWiki installation and make them writable to the web-server user: `/cache`, `/logs`, `/uploads`.
* Optional 3: For the [Linked Data](http://linkeddata.org/) plug-in to work, copy the `htaccess-dist` file to `.htaccess` make sure your Apache's `mod_rewrite` module is installed and `AllowOverwrite` is set to `All` for your OntoWiki directory in your `httpd.conf`.

### FAQ
* SQL Error:
[unixODBC][OpenLink][Virtuoso iODBC Driver][Virtuoso Server]FA011: Access to /var/www/ontowiki/libraries/Erfurt/include/SysOntLocal.rdf is denied due to access control in ini file (37000)
**Solution:** do Configuring Virtuoso Step 4
* Uncaught exception
    'Zend_Cache_Exception' with message 'Cannot use SQLite storage because the 'sqlite' extension is not loaded in the current PHP
**Solution:** go to ibraries/Erfurt/config/config.ini and set cache.enable = 0

### Platform-specific Notes

#### GNU/Linux
To start Virtuoso, switch to `<prefix>/bin` and run:
    ./virtuoso-t -f -c /opt/virtuoso/var/lib/virtuoso/ontowiki/virtuoso.ini

(Virtuoso runs in foreground using the created OntoWiki specific configuration file.)

Cave: Run Virtuoso with enough rights. When importing knowledge bases from file, they are first copied to tmp. Virtuoso needs a sufficiently high user like root or www-data to read it (besides adding the tmp dir to DirsAllowed). 
Otherwise you will get the error:
```
    Error importing knowledge base: Graph '<http://inserttesst.org>' could not be imported: Error importing statements: SQL Error: [unixODBC][OpenLink][Virtuoso iODBC Driver][Virtuoso Server]FA012: Can't open file '/tmp/phpWH20k0', error (13) : Permission denied (37000) CALL DB.DBA.RDF_LOAD_RDFXML(FILE_TO_STRING_OUTPUT('/tmp/phpWH20k0'), '', 'http://inserttesst.org'
```

##### Debian/Ubuntu
If you install Virtuoso from the .deb-Package you have to change in '/etc/init.d/virtuoso-opensource' the line
    DBBASE=/var/lib/virtuoso/db
to
    DBBASE=/var/lib/virtuoso/ontowiki
while virtuoso is not running.

Also, make sure your Virtuoso is at least [version 6.1.4]({{ "Deployment-Recommendations.html" | prepend:site.baseurl }})(http://docs.ontowiki.net/ontowiki_Deployment-Recommendations). **Virtuoso 7 is recommended, but for this you need a 64Bit System and 64Bit PHP**

**IMPORTANT HINT:** Make sure that you activated **display_errors** (=On) in your php.ini. Please have a look into phpinfo and check whats the values of display_errors. If its off, go to /etc/php5/apache2filter/php.ini respectively /etc/php5/apache2/php.ini and set the value on your own. Otherwise you will not see the error above!

#### Mac OS X
To auto-start Virtuoso on Mac OS X 10.5 use a config file for the `launchd` service. It should be placed under `/Library/LaunchDaemons`. The following is an example file, replace the paths with your settings where necessary.

    $ cat ~/Library/LaunchDaemons/com.openlinksw.virtuoso.plist 
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>UserName</key>
        <string>root</string>
        <key>GroupName</key>
        <string>wheel</string>
        <key>RunAtLoad</key>
        <true/>
        <key>Label</key>
        <string>com.openlinksw.virtuoso</string>
        <key>OnDemand</key>
        <false/>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/local/bin/virtuoso-t</string>
            <string>-f</string>
            <string>-c</string>
            <string>/usr/local/var/lib/virtuoso/ontowiki/virtuoso.ini</string>
        </array>
    </dict>
    </plist>

#### Windows
Follow [these instructions](http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSUsageWindows) in order to set up Virtuoso as an ODBC data source on Windows. Although a [binary distribution](http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSDownload#Pre-built binaries for Windows) of Virtuoso might seem the right choice, these are lagging a bit behind of the current source distribution. At the time of writing this line, Virtuoso is available in version 6.1.5, but the binary download for Windows is in version 6.1.3. OntoWiki works best with version 6.1.4 or greater. __It is important to note, that as of Version 7 Virtuoso no longer supports 32Bit ->To get it to work you need to use 64Bit Versions of Windows,Apache,PHP, otherwise you will get Architecture Mismatch errors especially für ODBC__

1. Apache

    1. Download Apache 2

        Go to `http://httpd.apache.org/download.cgi` and choose the latest stable release version that provides Windows binaries. Download the MSI Installer for this version. The line should look like _"Win32 Binary including OpenSSL 0.9.8t (MSI Installer): httpd-2.2.22-win32-x86-openssl-0.9.8t.msi"_. (Make sure to use the correct architecture)

    2. Run the Apache Installer

        When you arrive at the "Server Information" dialog box, enter `localhost` for the Network Domain as well as for the Server Name and whatever email address you wish for the "Administrator's Email Address" field. The installer uses the information you enter to create a default Apache configuration file for you. You can always go back and manually change these values in your configuration file if you change your mind later. Leave the default setting of "for All Users, on Port 80, as a Service" as it is. Click "Next" when you're done (see [How to Install and Configure Apache 2 on Windows](http://www.thesitewizard.com/apache/install-apache-2-windows.shtml)).   
Go to `http://localhost/` and confirm that it shows "It works!".
2. PHP
    - Stop the Apache service
        - Type `services.msc` into the search field in your start menu and click on "services"
        - Rightclick on "Apache2._X_" and click on "Stop"          
    - Go to <http://sourceforge.net/projects/phpinstallermsi/files/latest/download> and execute the MSI file that will automatically be downloaded.
    - Choose the default options presented by the wizard.
    - When prompted "Select a Web Server Setup" choose "Other CGI"
    - Add the following lines to the file `C:\Program Files\Apache Software Foundation\Apache2.2\httpd.conf`: (you may need to change the owner of that file to the current user in order to modify it) (see <http://windows.fyicenter.com/73_Apache_PHP_Getting_HTTP_403_Forbidden_Error_on_PHP_Scripts.html>)

            ScriptAlias /php/  "C:/Program Files/PHP/"
            AddHandler x-httpd-php .php
            Action x-httpd-php "/php/php-cgi.exe"
    
            <Directory "C:/Program Files/PHP/">
                AllowOverride None
                Options None
                Order allow,deny
                Allow from all
            </Directory>
        - search for `<Directory />` and change the contents of the tag to:

                <Directory />
                    Options All
                    AllowOverride All
                </Directory>

        - search for `/htdocs">` and change the the Directory tag to:

                <Directory "C:/Program Files/Apache Software Foundation/Apache2.2/htdocs">
                 Options Indexes FollowSymLinks               
                 Order allow,deny
                 Allow from all
                 AllowOverride All
                </Directory>
        - also uncomment the line `LoadModule rewrite_module modules/mod_rewrite.so`
    - Set the [recommended php.ini settings]({{ "php.html" | prepend:site.baseurl }}) in `C:\Program Files\PHP\php.ini`
    - Start the Apache service again
        - Type `services.msc` into the search field in your start menu and click on "services".
        - Rightclick on "Apache2._X_" and click on "Start".
    - Confirm that PHP works and is successfully integrated in Apache.
        - create the file `C:\Program Files\Apache Software Foundation\Apache2.2\htdocs\test.php` and set its content to `<?php phpinfo(); ?>`
        - Go to `http://localhost/test.php` and confirm that it shows a big table of PHP settings.
3. Virtuoso
    - Go to <http://www.openlinksw.com/dataspace/dav/wiki/Main/VOSDownload#Pre-built%20binaries%20for%20Windows> and choose "64-bit" if you use a 64-bit Windows or "32-bit" if you use a 32-bit Windows (you can determine it in "System Control Panel"->"System"->"System"->"System Type")
    - Unpack the directory `virtuoso-opensource` into the folder `C:\Program Files\`
  - Go to `C:\Program Files\virtuoso-opensource\database\virtuoso.ini` and set `DirsAllowed = ., ../vad,C:\Program Files\Apache Software Foundation\Apache2.2\htdocs\AKSW-OntoWiki-9c50d0e`
    - Follow [Using Virtuoso Open-Source Edition on Windows](http://virtuoso.openlinksw.com/dataspace/dav/wiki/Main/VOSUsageWindows) (follow "Creating a Windows Service for the Default Database" and "ODBC Driver Registration")
    - Follow [Virtuoso Driver for ODBC - Windows ODBC Driver Configuration](http://docs.openlinksw.com/virtuoso/odbcimplementation.html#virtdsnsetup), create a System DSN and name it "VOS".
    - Create the file `C:\Program Files\Apache Software Foundation\Apache2.2\htdocs\odbctest.php` with the following PHP code in it:

            <?php
            $conn   = odbc_connect('VOS', 'dba', 'dba');
            $query  = 'SELECT DISTINCT ?g WHERE {GRAPH ?g {?s ?p ?o.}}';
            $result = odbc_exec($conn, 'CALL DB.DBA.SPARQL_EVAL(\'' . $query . '\', NULL, 0)');
            ?>
            <ul>
            <?php while (odbc_fetch_row($result)): ?>
                <li><?php echo odbc_result($result, 1) ?></li>
            <?php endwhile; ?>
            </ul>
    - Go to <http://localhost/odbctest.php>. You should see this list of graphs stored in your Virtuoso RDF store:

            http://www.openlinksw.com/schemas/virtrdf#
            http://localhost:8890/sparql
            http://localhost:8890/DAV/
            http://www.w3.org/2002/07/owl#
If you see this list and no error messages along the way, go ahead configuring OntoWiki.
4. OntoWiki
    - Download [the newest version of OntoWiki from github](https://github.com/AKSW/OntoWiki/releases)
( choose "Download as zip").
    - Unpack the archive into the folder `C:\Program Files\Apache Software Foundation\Apache2.2\htdocs\`. You should now have the folder `...\htdocs\AKSW-OntoWiki-9c50d0e` (the last 7 characters may vary), from now on called %ONTOWIKI_HOME%.
    - Copy `%ONTOWIKI_HOME%\config.ini.dist` to `%ONTOWIKI_HOME%\config.ini`
    - Edit `%ONTOWIKI_HOME%\config.ini` and change the value of `store.backend` to "virtuoso"
    - Use `make install` in the root of your OntoWiki folder (you might need ton install some kind of port for GNU software tools like Cygwin or GnuWin32)
    - create the folder `...\AKSW-OntoWiki-9c50d0e\cache` and ensure that the user which runs Apache (System, if Apache is started as a service) has write access to that folder
    - Go to <http://localhost/AKSW-OntoWiki-9c50d0e/index.php> (adjust the URL if necessary). OntoWiki should now start.

#### Arch Linux

1. Apache, PHP and ODBC
    - Install Apache and PHP with the package manager

            $ sudo pacman -S apache php php-apache php-odbc
    - Configure PHP by following the PHP section of the [LAMP entry in the Archlinux Wiki](https://wiki.archlinux.org/index.php/LAMP#PHP) (also consult that page if you have any problems installing Apache and PHP)
    - Compile the libraries

            AKSW-OntoWiki-062a14e$ sudo make deploy
    - Activate the iconv and ODBC extensions in `/etc/php/php.ini`

        Uncomment the following lines:

            extension=iconv.so
            extension=odbc.so
    - Install ODBC

            $ sudo pacman -S php-odbc

        Add the following lines to the file `/etc/odbcinst.ini`: (create it if it doesn't exist, see [VirtuosoBackend]({{ "VirtuosoBackend.html" | prepend:site.baseurl }}))

            [virtuoso-odbc]
            Driver = /usr/lib/virtodbc.so

        Add the following lines to the file `/etc/odbc.ini`: (create it if it doesn't exist)

            [ODBC Data Sources]
            VOS = Virtuoso
        
            [VOS]
            Driver = virtuoso-odbc
            Description = Virtuoso Open-Source Edition
            Address = localhost:1111
    - (Re-)start Apache

            $ sudo /etc/rc.d/httpd restart
2. Virtuoso
    - Please note that Virtuoso-opensource should be at least in [version 6.4]({{ "Deployment-Recommendations.html" | prepend:site.baseurl }})
    - Install Virtuoso with the package manager

            $ sudo pacman -S virtuoso
    - Create the Virtuoso OntoWiki directory and add the virtuoso.ini to it (see [Virtuoso Backend]({{ "VirtuosoBackend.html" | prepend:site.baseurl }}))

            $ sudo mkdir /var/lib/virtuoso/ontowiki
            $ sudo cp /var/lib/virtuoso/db/virtuoso.ini /var/lib/virtuoso/ontowiki
    - Add the temporary and OntoWiki directories to the _DirsAllowed_ entry in `/var/lib/virtuoso/ontowiki/virtuoso.ini`

            DirsAllowed  = ., /usr/share/virtuoso/vad,/tmp,/srv/http/AKSW-OntoWiki-062a14e
    - Start Virtuoso (add the option _+foreground_ if you want to check if it starts correctly)

            $ sudo virtuoso-t -f -c /var/lib/virtuoso/ontowiki/virtuoso.ini

3. OntoWiki

either
    - [Download OntoWiki](https://github.com/AKSW/OntoWiki/releases) (choose "Download as tar.gz")
    - Unpack OntoWiki into your document root (the end of the file name may differ)
     
        `$ sudo tar -xzf download/AKSW-OntoWiki-v0.9.6-21-367-g062a14e.tar.gz --directory /srv/http`
or
    - get the newest (maybe unstable) Version with 
        
        `$ git clone https://www.github.com/AKSW/OntoWiki`
and then
    - Allow user read and write access to OntoWiki

     `sudo chmod a+wr -R /srv/http/AKSW-OntoWiki-062a14e`
    - rename `config.ini.dist` to `config.ini`
    - use `make install` 
    - Open http://localhost
The OntoWiki should now be shown after selection of the folder `AKSW-OntoWiki-#somenumber`

### Known Problems/Troubleshooting

#### Debian/Ubuntu
If you want to install virtuoso from the .deb and have installed virtuoso before (for instance on an update) it hast to run and has to be configured correctly during the installation, because the pre/pos-removal script tries to start/stop virtuoso.

#### In case OntoWiki isn't loaded correctly after you followed this tutorial
Go to `...\htdocs\elds\config.ini` and set `debug = true`. After a restart you should now see an error message in your browser which should hopefully point you to the source of the problem (if not, [create an issue](https://github.com/AKSW/OntoWiki/issues/new)). TODO: change to new url

#### Error on bootstrapping application: Unable to connect to Virtuoso Universal Server via ODBC
Make sure that the Virtuoso service is started. If it does not start, look for files named `virtuoso.lck` in your Virtuoso folder under `database` or `virtuoso` and delete them if existing.

#### The specified DSN contains an architecture mismatch between the Driver and Application
If you are using Windows, it is important to use the same architecture (32bit or 64bit) across all packages. Especially PHP-ODBC can only connect to Virtuoso if it is using the same architecture.
