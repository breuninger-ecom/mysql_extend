mysql_extend
============

"MySQL-Monitoring-Proxy-Tool" for Zabbix

This tool can be utilized to gather behavioral measurement data or configuration data of MySQL servers in a efficient way. 
It can be used as an external check for the zabbix monitoring system.
The benefit for using this tool is a very low overhead for gathering the measurements because this tool is written in c and uses
caching of values.

MySQL Measurements and Configuration details:
 * "SHOW GLOBAL VARIABLES"
 * "SHOW GLOBAL STATUS"
 * "SHOW SLAVE STATUS"
 * "SHOW MASTER STATUS"

What it does in general:
 * On invocation the tool creates a lock which blocks all other running instances of this tool
 * It searches for a shared memory segment which is used for caching measurement values
   * If the segment does not exist a new segment is created by the tool
   * A lookup is performed to find out if there are already cached values for the specified host
   * If the stored measures are not older than 60 seconds the specified measure is provided to stdout and the program flow is terminated
 * If the store values are older than 60 seconds or not stored values are available an database connection is established and the measurements listed above are executed
   * The results are stored in the shared memory segment (if there are any) with the timestamp of the fetch
   * If no results are available the old results remain in the shared memory segment
 * End of program flow, release the lock to allow other instances of the tool to continue their work

# Compile and Install

## Prerequisites:
 * C-compiler
 * GNU Make
 * MySQL-Client libraries (dev)

## Build and Installation

This will install the binary 'mysql_extend' to /usr/local/bin:
```
sudo apt-get install autoconf automake libmysqlclient-dev
git clone git@github.com:breuninger-ecom/mysql_extend.git
cd mysql_extend
export CC="gcc-4.4" # alternative compiler, i.e. on Ubuntu 13.10
export LDFLAGS="-lrt"
./configure --bindir=/etc/zabbix/externalscripts/ # Installdir of tool
make
make install
```

## Bugs

- fix mysql 5.6/mariadb 5.5. problems which "SHOW GLOBAL VARIABLES"
  After "optimizer_switch" output seems to be broken.
  (see sql.c => search for "optimizer_switch")

## Configure
---------

* create a database user

  ```
grant usage, replication client on *.* to monitor@'%' identified by 'somegoodpassword';
FLUSH privileges;
  ```

* create a .my.cnf in the zabbix-user's home:


  ```
[mysql_extend]
user = monitor
password = somegoodpassword
  ```

# Usage

Configure items like this in zabbix:

```
Description........: MySQL Com_alter_function
Type...............: External check
Key................: mysql_extend[Com_alter_function -P3306]
Type of information: Numeric (unsigned)
Data type..........: Decimal
Update interval....: 300
Store value........: Delta (simple change)
```

Call the tool with parameters like this:

```
/usr/local/bin/mysql_extend -P 3306 -t 2 Binlog_cache_use foo.bar.de
/usr/local/bin/mysql_extend --help
```

The directory "zabbix" contains a example zabbix monitoring template which measures and notifies.

Installation:
 * Install "mysql_extend" on the zabbix-server or zabbix-proxy (make install)
 * Import the zabbix-template to your zabbix-server
 * Assign hosts to the template and overload the macros of the template at host level.

Review file "zabbix/Custom_-_Service_-_MySQL.html" to get detailed information about the behavior of this template.
[see also](http://htmlpreview.github.io/?https://github.com/digitalmediacenter/mysql_extend/blob/master/zabbix/Custom_-_Service_-_MySQL.html)

# Measurement details

## "SHOW GLOBAL VARIABLES"

http://dev.mysql.com/doc/refman/5.6/en/show-variables.html

All measurements provided by the statement.

Key: column "Variable_name"
result: column "Value"
 
## "SHOW GLOBAL STATUS"

http://dev.mysql.com/doc/refman/5.6/en/server-status-variables.html

All measurements provided by the statement.

Key: column "Variable_name"
result: column "Value"

## "SHOW SLAVE STATUS"

http://dev.mysql.com/doc/refman/5.6/en/show-slave-status.html

Key: Slave_IO_Running
result: column "Slave_IO_Running"

Key: Slave_SQL_Running
result: column "Slave_SQL_Running

Key: Seconds_Behind_Master
result: column "Seconds_Behind_Master

## "SHOW MASTER STATUS"

http://dev.mysql.com/doc/refman/5.6/en/show-master-status.html

Key: Position
result: column "Position"

# Licence and Authors


Additional authors are very welcome - just submit you patches as pull requests.

 * Andreas Heil <andreas.heil@dmc.de>
 * Marc Schoechlin <ms@256bit.org>
 * Marc Schoechlin <marc.schoechlin@breuninger.de>
 * Marc Schoechlin <marc.schoechlin@breuninger.de>

License - see: LICENSE.txt
