zfs-repl
========
A multi-platform ZFS snapshot and replication tool supporting compression and encryption over multiple transport protocols.

Purpose
=======
This tool/script will create and replicate ZFS datasets/file systems using snapshots between source and target hosts on *nix systems.  Its intended to be run primarily via cron as the root user but can be run by other users (with appropriate privileges) from the command line.

The tool has been written to be flexible so that it can run on and against hopefully any *nix system that supports ZFS storage pools.

This system has been tested on the following platforms:
* Ubuntu 16.04, 18.04, 20.04


Background/History
==================
This script is a fork/modification of others prior work.  Specifically, the work of 
- Mike La Spina
- srlefevre

Prerequisites
===========
1. You know what your doing 
   * You know how to setup ssh to use public/private key authentication 
   * You know how to setup a user under sudo to not prompt for a password
2. Both source and target systems are running ZFS pools
3. User on source system is root or can run sudo.
4. User on target is root or can run sudo without a password for the zfs command.
5. Source system has a functioning ssh client.
6. Target systems have a functioning ssh server.
7. The user on source system can ssh to target system using public/private keys without prompting for a password (e.g. 'ssh repl-user@target')
8. Source system has GNU date command installed.  (This is needed to handle aging/deletion of snapshots)

Setup
=====
For those that pace in front of microwave ovens...

Install
-------

All commands should be executed as root or with sudo.

download in any place
```
cd /opt
git clone https://github.com/kattunga/zfs-repl.git
```

Configure
---------

Create the configuration file and then edit the configuration file using your preferred text editor.

```
./source-config
nano zfs-repl.conf  
```

Optionally, if target host is a different platform/os version, you shoud create a custom target configuration file

```
./target-config --host [[user@]target-name] > hosts/[target-name].conf
```

zfs-repl.conf
-------------
Global config file


/hosts/[target-name].conf
-------------------------
*Note:* For the uninitiated, [target-host] should be replaced by the hostname of your target system. For example, if you specify --host repl@nas12 on the 'zfs-repl' command line then nas12 would be your hostname and the configuration file would be /etc/zfs-repl/nas12.conf

The /etc/zfs-repl/[target-name].conf file holds configuration setting specific to the target host. The basic setup should be completed using the 'target-config' command as stated earlier.  It can also be used to set defaults for the specific host.  

If you need to replicate to more then one host, make sure you run the 'target-config' command for each host and create the appropriate .conf file for each host.  For example:

```
sudo su
./target-config --host repl@host1 > hosts/host1.conf
./target-config --host root@host2 > hosts/host2.conf
exit
```


Usage
=====

See 'zfs-repl --help'.

Example
-------
Source system hostX needs to backup the every changing file system pool0/data01 to the target system hostZ under tank1/backup/data01.  hostX wants to maintain a rolling two days of snapshots but hostZ needs to maintain seven days of snapshots.  Further, hostZ needs to compress the backup file system using lz4.

On hostZ, at least zpool tank1 should already be created but tank1/backup/data01 should not.

```
/opt/zfs-repl/zfs-repl --host repl@hostZ --source pool0/data01 --dest tank1/backup/data01 --snap-retain "-2 days" --dest-snap-retain "-7 days" --create-dest --dest-opt compression=lz4
```

The above command could be added to crontab with required schedule.

Notes
=====

mbuffer
-------

mbuffer is available on many *nix distrobutions as part of the base operating system.  On Red Hat EL6/CentOS 6 and other derived linux distrobutions, it is not included. I was able to find mbuffer from Fedora Core 13 works on CentOS 6 without issue.


