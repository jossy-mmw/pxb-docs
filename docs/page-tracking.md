# Take an incremental backup using page tracking

*Percona XtraBackup* 8.0.27 adds support for page tracking functionality for an [incremental backup](create-incremental-backup.md).

To create an incremental backup with page tracking, Percona XtraBackup uses
the MySQL `mysqlbackup` component. This component provides a list of pages
modified since the last backup, and Percona XtraBackup copies only those
pages. This operation removes the need to scan the pages in the
database. If the majority of pages have not been modified, the page
tracking feature can improve the speed of incremental backups.

## Install the component

To start using the page tracking functionality, do the following:

1. Install the `mysqlbackup` component and enable it on the server:

    ```{.bash data-prompt="$"}
    $ INSTALL COMPONENT "file://component_mysqlbackup";
    ```

2. Check whether the `mysqlbackup` component is installed successfully:

    ```{.bash data-prompt="$"}
    $ SELECT COUNT(1) FROM mysql.component WHERE component_urn='file://component_mysqlbackup';
    ```

## Use page tracking

You can enable the page tracking functionality for the full and incremental
backups with the `--page-tracking` option.

The option has the following benefits:

* Resets page tracking to the start of the backup. This reset allows the next incremental backup to use page tracking.

* Allows the use of page tracking for an incremental backup if the page tracking data is available from the backup’s start checkpoint LSN.

*Percona XtraBackup* processes a list of all the tracked pages in memory. If *Percona XtraBackup* does not have enough available memory to process this list, the process throws an error and exits. For example, if an incremental backup uses 200GB, *Percona XtraBackup* can use an additional 100MB of memory to process and store the page tracking data. 

The examples of creating full and incremental backups using the `--page-tracking` option:

=== "Full backup"

    ```{.bash data-prompt="$"}
    $ xtrabackup --backup --target-dir=$FULL_BACK --page-tracking
    ```

=== "Incremental backup"

    ```{.bash data-prompt="$"}
    $ xtrabackup --backup --target-dir=$INC_BACKUP  
    --incremental-basedir=$FULL_BACKUP --page-tracking
    ```

After enabling the functionality, the next incremental backup finds changed
pages using page tracking.

### Limitations

* When creating the first full backup using page tracking, Percona XtraBackup may have a delay. The following is an example of the message:

    ```{.text .no-copy}
    xtrabackup: pagetracking: Sleeping for 1 second, waiting for checkpoint lsn 17852922 /
    to reach to page tracking start lsn 21353759
    ```

    Enable page tracking before creating the first backup to avoid this delay. This method ensures that the page tracking log sequence number (LSN) is higher than the checkpoint LSN of the server.

* If an Oracle MySQL Server uses page tracking and runs `LOCK=EXCLUSIVE` and `ALGORITHM=INPLACE` DDL operations, Percona XtraBackup may take an inconsistent backup. Oracle MySQL Server does not write these DDL operations (Oracle MySQL [bug 106163](https://bugs.mysql.com/bug.php?id=106163)) to the redo log, and Percona XtraBackup only detects the inconsistency during the `--prepare` phase.

    This bug does not influence Percona XtraBackup taking backup of Percona Server for MySQL because the bug is fixed in Percona Server for MySQL 8.0.27.

* The xtrabackup page tracking requires the system tablespace to be a single file (ibdata1 only). The page tracking feature does not work if the system tablespace is a multi-file tablespace.

## Start page tracking manually

After the mysqlbackup component is loaded and active on the server, you can
start page tracking manually with the following option:

```{.bash data-prompt="$"}
$ SELECT mysqlbackup_page_track_set(true);
```

## Check the LSN value

Check the LSN value starting from which changed pages are tracked with the
following option:

```{.bash data-prompt="$"}
$ SELECT mysqlbackup_page_track_get_start_lsn();
```

## Stop page tracking

To stop page tracking, use the following command:

```{.bash data-prompt="$"}
$ SELECT mysqlbackup_page_track_set(false);
```

## Purge page tracking data

When you start page tracking, it creates a file under the server’s datadir
to collect data about changed pages. This file grows until you stop the
page tracking. If you stop the server and then restart it, page tracking
creates a new file but also keeps the old one. The old file continues to
grow until you stop the page tracking explicitly.

If you purge the page tracking data, you should create a full backup
afterward. To purge the page tracking data, do the following steps:

```{.bash data-prompt="$"}
$ SELECT mysqlbackup_page_track_set(false);
$ SELECT mysqlbackup_page_track_purge_up_to(9223372036854775807);
/* Specify the LSN up to which you want to purge page tracking data. /
9223372036854775807 is the highest possible LSN which purges all page tracking files.*/
$ SELECT mysqlbackup_page_track_set(true);
```

## Known issue

If the index is built in place using an exclusive algorithm and then is
added to a table after the last LSN checkpoint, you may generate a bad
incremental backup using page tracking. For more details
see [PS-8032](https://jira.percona.com/browse/PS-8032).

## Uninstall the mysqlbackup component

To uninstall the mysqlbackup component, use the following statement:

```{.bash data-prompt="$"}
$ UNINSTALL COMPONENT "file://component_mysqlbackup"
```
