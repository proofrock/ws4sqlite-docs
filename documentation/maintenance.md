# 🔨 Maintenance

Going back to this snippet of [the configuration file](configuration-file.md):

```yaml
maintenance:
  schedule: 0 0 * * *
  atStartup: false
  doVacuum: true
  doBackup: true
  backupTemplate: ~/first_%s.db
  numFiles: 3
  statements:
    - DELETE FROM myTable WHERE tstamp < CURRENT_TIMESTAMP - 3600
    - ... 
```

The `maintenance` node represent the structure that tells ws4sqlite to provide scheduled maintenance. This maintenance
can be:

- scheduled, with a cron-like syntax; and/or
- performed at startup.

The maintenance itself can be comprised of one or more of:

- a VACUUM, to optimize and cleanup the internal structures;
- a backup, rotated as needed;
- a set of SQL statements, for example to cleanup temporary tables.

The last feature in particular is very powerful, in that it allows to perform statements at startup, or repeatedly; if
for example you need to generate a sort of "run id" for one particular run, the relevant SQL can be executed at each
startup of the server.

We'll now discuss the configurations.

#### `schedule`

_Line 2; string; mandatory this or `atStartup` as true_

Cron-like string, standard 5-fields (no seconds). See [documentation](https://www.adminschoice.com/crontab-quick-reference) for more details.

#### `atStartup`

_Line 3; boolean; mandatory this as true or `schedule`_

If true, performs a maintenance routine at engine startup.

{% hint style="info" %}
It is possible that ws4sqlite is started two times in the same minute; in this case, the backup file _won't be overwritten_.
{% endhint %}

#### `doVacuum`

_Line 4; boolean_

If present and set to `true`, performs a [`VACUUM`](https://www.sqlite.org/lang\_vacuum.html) on the database.

#### `doBackup`

_Line 5; boolean_

If present and set to `true`, takes a snapshot of the database.

If `doVacuum` is also `true`, the backup is taken after the `VACUUM`.

The backup is created with the `VACUUM INTO...` command.

The following parameters tell ws4sqlite how to manage the backup(s).

#### backupTemplate

_Line 6; string; mandatory if `doBackup` is `TRUE`_

Filename (with path) of the backup files. It must contain a single `%s` that will be replaced with the minute of the 
backup, in `yyyyMMdd-HHmm` format. For example:

`../myFile_%s.db` will be generated as `../myFile_20220127-1330.db`

`~` is expanded to the user's home directory path.

{% hint style="warning" %}
If the same pattern is specified for different databases (it doesn't make much sense, but still) it is possible that
ws4sqlite generates the same backup file more than once. In this case, the first backup file "wins" and _won't be
overwritten_. The order of operations is deterministic but unspecified; so it's much better to avoid the situation.
{% endhint %}

#### numFiles

_Line 7; number; mandatory if `doBackup` is `TRUE`_

Indicates how many files to keep for each database. After the limit is reached, the files rotate, with the least recent files being deleted.

### statements

_Lines 8-10; list of strings_

A set of SQL statements (without parameters) to execute. 

{% hint style="warning" %}
The statements are not run in a transaction: if one fails, the next one will be executed, as with `initStatements`. On
the other hand, there is a mutex that ensures that the statements' block is not executed concurrently to a request.
{% endhint %}
