Install MySQL on Mac OS X
=============================

Notation:

    BASEDIR == <install_folder>


Steps
--------------

Download the tar.gz archive and unpack.

    $ cd BASEDIR/
    $ unset TMPDIR
    $ ./scripts/mysql_install_db

Modify `BASEDIR/support-files/mysql.server` file and set some properties:

    basedir = BASEDIR
    datadir = BASEDIR/data

Start the server (from BASEDIR):

    $ ./support-files/mysql.server start

Secure the installation (from BASEDIR):

    $ ./bin/mysql_secure_installation

Stop the server (from BASEDIR):

    $ ./support-files/mysql.server stop

Add environment variables to `.bash_profile` or `.zshenv`

    $ export MYSQL_HOME=<path_to_BASEDIR>
    $ export PATH=$MYSQL_HOME/bin:$PATH