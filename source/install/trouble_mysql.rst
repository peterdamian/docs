.. _mysql-installation-troubleshooting:

MySQL Installation Troubleshooting
----------------------------------

Before you can run the Mattermost server, you must first install and
configure a database. If the Mattermost server cannot connect to the database, Mattermost
will fail to start. This section deals with MySQL database issues that
you may encounter when you start up Mattermost for the first time.

How you install MySQL varies depending upon which Linux distribution you
use, but once MySQL is installed, the configuration instructions are the
same for all supported distributions. You must create a ``mattermost`` database
and a ``mattermost`` database user. Failure to create these database
objects or improperly referencing them from the Mattermost configuration
file prevents Mattermost from starting. The following troubleshooting tips deal with these specific
issues.

Before proceeding, confirm that your MySQL server is running. You can do
this by issuing the following command: ::

    mysqladmin -u root -p status

When prompted, enter your password. If MySQL is running you should see output
like the following: ::

    Uptime: 877134  Threads: 1  Questions: 9902  Slow queries: 0  Opens: 522  
    Flush tables: 1  Open tables: 371  Queries per second avg: 0.011

If MySQL is not running, review the instructions for installation on
your distribution.

.. warning:: Some of the commands used in this section alter the database. **Use these commands only if your Mattermost installation has failed. Do not directly manipulate the MySQL database for a working Mattermost installation**.

The Database
~~~~~~~~~~~~~~~~~~~~~~~

The database created during installation should be named ``mattermost``. If you
fail to create this database or you misname it, you will see the following error 
when attempting to start the Mattermost server: ::

    [2017/09/20 17:11:37 EDT] [INFO] Pinging SQL master database
    [2017/09/20 17:11:37 EDT] [EROR] Failed to ping DB retrying in 10 seconds
    err-Error 1044: Access denied for user 'mmuser'@'%' to database 'mattermost'

Note that MySQL is specifically denying access to the ``mattermost``
database. This may mean that you have failed to create a database named
``mattermost`` or you may have incorrectly referenced this database from
the ``config.json`` file.

**Checking that the Database Exists**

To confirm that the ``mattermost`` database exists, open MySQL as root
by executing ``mysql -u root -p``. When prompted, enter your
password and then issue the following command: ::

    show databases;

This command displays all the databases. You should see something similar to the
following: ::

    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mattermost         |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    5 rows in set (0.03 sec)

**No Database**

If the ``mattermost`` database doesn't exist, create a database named
``mattermost`` by opening MySQL as root and by issuing the command: ::

    create database mattermost;

If you accidentally created a database with the wrong name, you can
remove it by issuing the command: ::

    drop database {misnamed};

**The Database Exists**

If the ``mattermost`` database does exist, confirm that you have set up
the database driver correctly in the ``config.json`` file. Open this file in a text
editor, and review the value of ``"DataSource"``. It should be: ::

    "mmuser:<mmuser-password>@tcp(<host-name-or-IP>:3306)/mattermost?charset-utf8mb4,utf8&readTimeout-30s&writeTimeout-30s"

Replace `mmuser-password` with the password created during installation
and `host-name-or-IP` with the appropriate value.
You should also confirm that the ``DatabaseSettings.DriverName`` element (found immediately
above the ``DatabaseSettings.DataSource`` element) is set to ``mysql``.

The Database User
~~~~~~~~~~~~~~~~~

During installation, a MySQL database user is created from the *mysql*
prompt by issuing the following command:::

    create user 'mmuser'@'%' identified by '{mmuser-password}';`
    
The ``mmuser-password`` value is a placeholder for the password you chose.

.. note:: MySQL users are fully defined by their username and the host that they access MySQL from. These elements are separated by the ``@`` sign. The ``%`` character is a wild card indicating that the user can access MySQL from any IP address or host. If the user you created accesses MySQL from a specific host such as ``10.10.10.2``, please substitute that IP address for ``%``.

If the user and host combination that you created do not exist, you
will see an error such as: ::

    [2017/09/20 17:06:18 EDT] [INFO] Pinging SQL master database
    [2017/09/20 17:06:18 EDT] [EROR] Failed to ping DB retrying in 10 seconds 
    err-Error 1045: Access denied for user 'mmuser'@'localhost' (using password: YES)

**Checking that the Database User Exists**

To check that this user exists, log in to MySQL as the root user: ::

    mysql -u root -p

When prompted, enter the root password that you chose 
when installing MySQL. From the ``mysql`` prompt enter the following command: ::

    select User, Host from mysql.user where User = 'mmuser';

You should see the following if the user exists: ::

    +------------------+-----------+
    | User             | Host      |
    +------------------+-----------+
    | mmuser           | %         |
    | debian-sys-maint | localhost |
    | mysql.session    | localhost |
    | mysql.sys        | localhost |
    | root             | localhost |
    +------------------+-----------+
    5 rows in set (0.00 sec)

**User Doesn't Exist**

If ``'mmuser'@'%'`` does not exist, create this user by logging into
MySQL as root and by issuing the command: ::

    create user 'mmuser'@'%' identified by '{mmuser-password}';

After creating a user, ensure that this user has rights to the
``mattermost`` database by following the instructions given in
:ref:`mysql_grants`.

**User Exists**

If the ``mmuser`` user exists, the ``DataSource`` element of the
``config.json`` file may be incorrect. Open this
file and search for ``DataSource``. Its value should be: ::

    "mmuser:<mmuser-password>@tcp(<host-name-or-IP>:3306)/mattermost?charset-utf8mb4,utf8&readTimeout-30s&writeTimeout-30s"

The password `mmuser-password` is a placeholder for the
password you created during installation.

The User Password
~~~~~~~~~~~~~~~~~

Mattermost will fail if you use an incorrect password for ``mmuser``. An
incorrect password displays an error message such as the following: ::

    [2017/09/20 17:09:10 EDT] [INFO] Pinging SQL master database
    [2017/09/20 17:09:10 EDT] [EROR] Failed to ping DB retrying in 10 seconds 
    err-Error 1045: Access denied for user 'mmuser'@'localhost' (using password: YES)

**The Password in config.json**

The ``DataSource`` element of the ``config.json``
file references the ``mmuser`` password. Open this file and search for
``DataSource``. Its value should be: ::

    "mmuser:<mmuser-password>@tcp(<host-name-or-IP>:3306)/mattermost?charset-utf8mb4,utf8&readTimeout-30s&writeTimeout-30s"

Ensure that the password you used in place of ``mmuser-password``
is correct.

**Unsure of Password**

If you are not sure that the ``mmuser`` password is correct, attempt to
log in to MySQL as ``mmuser`` by issuing the command: ::

    mysql -u mmuser -p

You will be prompted for your password. If your
login fails, you are not using the correct password.

With a new database installation, the easiest solution for an unknown
password is to remove the existing ``mmuser`` and then recreate that
user. You do this by logging in to MySQL as root and by issuing the
following commands:

1. ``drop user mmuser;``

2. ``flush privileges;``

3. :samp:`create user 'mmuser'@'%' identified by '{mmuser-password}';`

If you recreate ``mmuser``, ensure that this user has rights to the
``mattermost`` database by following the instructions given in
:ref:`mysql_grants`.

.. _mysql_grants:

Insufficient User Privileges
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the database exists and the username and password are correct, the
``mmuser`` may not have sufficient rights to access the ``mattermost``
database. If this is the case, you may see an error message such as: ::

    [2017/09/20 17:20:53 EDT] [INFO] Pinging SQL master database
    [2017/09/20 17:20:53 EDT] [EROR] Failed to ping DB retrying in 10 seconds 
    err-Error 1044: Access denied for user 'mmuser'@'%' to database 'mattermost

.. note:: Examine the error message closely. The user name displayed in the error message is the user identified in the ``DataSource`` element of the ``config.json`` file. For example, if the error message reads ``Access denied for user 'muser'@'%' ...`` you will know that you have misidentified the user as ``muser`` in the ``config.json`` file.

You can check if the user ``mmuser`` has access to the ``mattermost``
database in the following way:

    1. Log in to MySQL as ``mmuser``.
    2. Issue the command: ``show databases;``. 
    
If ``mmuser`` does not have rights to view the
``mattermost`` database, the output will look similar to the following: ::

    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    +--------------------+
    1 rows in set (0.00 sec)
    
**Granting Privileges to the Database User**

If the ``mattermost`` database exists and ``mmuser`` cannot view it,
exit from MySQL and then log in again as root. Issue the command
``grant all privileges on mattermost.* to 'mmuser'@'%';`` to grant
all rights on ``mattermost`` to ``mmuser``.
