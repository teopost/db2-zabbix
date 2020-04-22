# DB2 Zabbix Monitoring Scripts

This project contains DB2 Zabbix monitoring scripts used as user parameters by Zabbix agent.

## Installation

The repository includes ready-to-install files for Zabbix Agent.

* Copy the files under [etc/zabbix/scripts](etc/zabbix/scripts) to `/etc/zabbix/scripts`
* Copy the files under [etc/zabbix/zabbix_agentd.d](etc/zabbix/zabbix_agentd.d) to `/etc/zabbix/zabbix_agentd.d`

## Templates

Import Zabbix template from [templates](templates) folder to Zabbix.

# DB2 Database Snapshot Statistics (db2stat)

This script generates database snapshots (i.e. get snapshot for database) from
DB2 and retrieves statistics from it.

Because DB2 install location varies by system and installation method, the path
to DB2 executable must be edited into PATH environment variable set up in the
script.

Zabbix agent user must also have permission to create database snapshots. See
below on how to do this.

## Enabling DB2 Snapshots for Zabbix User

To allow Zabbix agent user to create snapshots it must have capability in DB2
to do that. For monitoring purposes the best match is the SYSMON permission. The
operating system group for this is set via DB2 configuration parameters.

To enable snapshots:

1. Create db2mon group in operating system and add zabbix agent user to it (Zabbix agent must be installed so that zabbix user is present).

```
# Linux systems
groupadd db2mon
usermod -a -G db2mon zabbix

#AIX systems
mkgroup db2mon
chgrpmem -m + zabbix db2mon
```

2. Configure db2mon group have SYSMON permission in database execute following *as db2 user*:

```
# Configure db2mon group

# For db2stat zabbix metrics
db2 update dbm cfg using sysmon_group db2mon

# For lock waits zabbiz metrics
db2 "grant select on table SYSIBMADM.MON_LOCKWAITS to db2mon"

# Restart database
db2stop
db2start
```

To test taking the snapshot with zabbix user in Linux as root:

```
# Linux systems
su -s /bin/bash -c "export DB2INSTANCE=<instance-name>;/opt/ibm/db2/V10.5/bin/db2 get snapshot for database on <db>" zabbix

# AIX systems
su - zabbix -c "<path-to-db2dir>/bin/db2 get snapshot for database on <db>"`
```

## Installing Items from Template

Zabbix template for all items supported in configuration is [here](templates/db2stat.xml).
To configure it, at least macro value for DATABASE_NAME must be updated.

1. Step 1: Template import 
  * Menu `Configuration`
  * Submenu `Templates`
  * Click on button `Import` and select templates/db2stat.xml
  
2. Step 2: Assign template to an hosts
  * Menu `Configuraziont
  * Submenu Hosts
  * Select an host and click `Template`
  * Add `Template DB2` to linked template and confirm
  * Click on `Macro`
  * Click in `Inherited and host macros`
  * Configure DB2 parameters (example):
    * {$DATABASE_INSTANCE} = dbstgadm
    * {$DATABASE_NAME} = DBWCSTG
    * {$DATABASE_PATH} = /opt/ibm/db2/V10.5/bin <- Path of the db2 binary

3. Test and collect data  
  * Menu `Monitoring`
  * Submenu `Latest data`
  * Select an host
  * Check if there is any data
 
## Manual Item Configuration

Provided user parameter configuration contains several parameters. Consult the
[configuration file](../etc/zabbix/zabbix_agentd.d/db2stat.conf) for a full list.

Simple statistics can be retrieved with two parameters, maximum snapshot age in seconds and database:

`db2stat.database_status[60,SAMPLE]`

Retrieving memory statistics additionally requires node number:

`db2stat.package_cache_heap_size[60,SAMPLE,0]`

## Thanks to

db2-zabbix is a modified fork of https://github.com/digiapulssi/zabbix-monitoring-scripts
