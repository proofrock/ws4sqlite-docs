# 🔨 Maintenance

Going back to this snippet of[ the configuration file](configuration-file.md):

```yaml
maintenance:
  schedule: 0 0 * * *
  doVacuum: true
  doBackup: true
  backupTemplate: ~/first_%s.db
  numFiles: 3
```

The `maintenance` node represent the structure that tells ws4sqlite to provide scheduled maintenance. This means, a VACUUM or a backup - or both.

#### `schedule`

_Line 2; string; mandatory_

Cron-like string, standard 5-fields (no seconds). See [documentation](https://www.adminschoice.com/crontab-quick-reference) for more details.

#### `doVacuum`

_Line 3; boolean_

If present and set to `true`, performs a [`VACUUM`](https://www.sqlite.org/lang\_vacuum.html) on the database.

#### `doBackup`

_Line 4; boolean_

If present and set to `true`, takes a snapshot of the database.

If `doVacuum` is also `true`, the backup is taken after the `VACUUM`.

The backup is created with the `VACUUM INTO...` command.

The following parameters tell ws4sqlite how to manage the backup(s).

#### backupTemplate

_Line 5; string; mandatory if `doBackup` is `TRUE`_

Path of the backup files. It must contain a single `%s` that will be replaced with the minute of the backup, in `yyyyMMdd-HHmm` format. For example:

`../myFile_%s.db` will be generated as `../myFile_20220127-1330.db`

`~` is expanded to the user's home directory path.

#### numFiles

_Line 6; number; mandatory if `doBackup` is `TRUE`_

Indicates how many files to keep for each database. After the limit is reached, the files rotate, with the least recent files being deleted.
