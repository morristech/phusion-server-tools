# Phusion Server Tools

A collection of server administration tools that we use. Everything is
written in Ruby and are designed to work with Debian. These scripts may
work with other operating systems or distributions as well, but it's not
tested.

Install with:

    git clone https://github.com/FooBarWidget/phusion-server-tools.git /tools

It's not necessary to install to /tools, you can install to anywhere, but this document assumes that you have installed to /tools.

Each tool has its own prerequities, but here are some common prerequities:

 * Ruby (obviously)
 * `pv` - `apt-get install pv`. Not required but very useful; allows display of progress bars.


## Included tools

### backup-mysql - Rotating MySQL dumps

A script which backs up all MySQL databases to `/var/backups/mysql`. At most 10 backups are kept. All backups are compressed with gzip. The backup directory is denied all world access.

It uses `mysql` to obtain a list of databases and `mysqldump` to dump the database contents. If you want to run this script unattended you should therefore set the right login information in `~/.my.cnf`, sections `[mysql]` and `[mysqldump]`.

Make it run daily at 0:00 AM in cron:

    0 0 * * * /tools/silence-unless-failed /tools/backup-mysql

### permit and deny - Easily set fine-grained permissions using ACLs

`permit` recursively gives a user access to a directory by using ACLs. The default ACL is modified too so that any new files created in that directory or in subdirectories inherit the ACL rules that allow access for the given user.

`deny` recursively removes all ACLs for a given user on a directory, including default ACLs.

The standard `setfacl` tool is too hard to use and sometimes does stupid things such as unexpectedly making files executable. These scripts are simple and work as expected.

    # Recursively give web server read-only access to /webapps/foo.
    /tools/permit www-data /webapps/foo
    
    # Recursively give user 'deploy' read-write access to /webapps/bar.
    /tools/permit deploy /webapps/bar --read-write
    
    # Recursively remove all ACLs for user 'joe' on /secrets/area66.
    /tools/deny joe /secrets/area66

You need the `getfacl` and `setfacl` commands:

    apt-get install acl

You must also make sure your filesystem is mounted with ACL support, e.g.:

    mount -o remount,acl /

Don't forget to update /etc/fstab too.

### display-queue - Display statistics for local RabbitMQ queues

This tool displays statistics for RabbitMQ queues in a more friendly formatter than `rabbitmqctl list_queues`. The meanings of the columns are as follows:

 * Messages - Total number of messages in the queue. Equal to `Ready + Unack`.
 * Ready - Number of messages in the queue not yet consumed.
 * Unack - Number of messages in the queue that have been consumed, but not yet acknowledged.
 * Consumers - Number of consumers subscribed to this queue.
 * Memory - The amount of memory that RabbitMQ is using for this queue.

### watch-queue - Display changes in local RabbitMQ queues

`watch-queue` combines the `watch` tool with `display-queue`. It continuously displays the latest queue statistics and highlights changes.

### purge-queue - Removes all messages from a local RabbitMQ queue

`purge-queue` removes all messages from given given RabbitMQ queue. It connects to a RabbitMQ server on localhost on the default port. Note that consumed-but-unacknowledged messages in the queue cannot be removed.

    purge-queue <QUEUE NAME HERE>

### truncate

Truncates the all passed files to 0 bytes.

### silcence-unless-failed

Runs the given command but only print its output (both STDOUT and STDERR) if its exit code is non-zero. The script's own exit code is the same as the command's exit code.

    /tools/silence-unless-failed my-command arg1 arg2 --arg3
