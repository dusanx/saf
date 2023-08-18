# saf

"one backup is saf, two are safe, three are safer"

**saf** is a simple, reliable, rsync-based, battle tested, well rounded backup system, written in Python.

It uses rsync to incrementally back up your data to a different directory, hard disk or remote server via SSH. All operations are incremental, atomic and optionally possible to resume.

**saf** is using very limited set of commands in the background, so it plays well with storage shells with limited set of commands, like Hetzner Storage Box. 

Main backup code is used in production for many years and has proved its reliability, while recent **saf** updates only improve user facing commands. No guarantees but should be safe to use.

## Basic concepts

In **saf** every local file system directory can become backup source location. Each backup source location can have one or more backup target locations, in practice same directory (and its sub-directories) can be backed up to several target locations.

Each backup source location (initialised by `saf init`) contains `.saf.conf` file, which is a simple and easy to change configuration file in Python configparser format. It is easy to query backup source locations down the directory tree with `saf status` or `saf status --all`, or up the directory tree with `saf status --reverse]`.

Each `.saf.conf` can contain one or more `[<name>.target]` sections, defining backup target locations. It is easy to add any number of different local or remote backup locations with `saf init --name <backup-name> <target-location>`, where `target-location` can be local directory `/mnt/backup-disk/backup-path` or remote storage/server `my-server:/mnt/backup-disk/backup-path`.

Every **saf** command is location relative. **saf** will find `.saf.conf` in current or any of the parent directories and perform requested command. First backup target is always considered default but all the commands can specify what target name they are referring to.

> **_NOTE:_**  Don't use relative paths to specify target location. Always specify full target location `/home/user/backup` instead of `./backup` or `~/backup`. Relative locations are user related and not suitable for backup target.

> **_NOTE:_**  Final slash `/` when specifying target location doesn't matter, **saf** will interpret location with or without slash correctly.

Every `.saf.conf` contains `[exclude]` section that uses standard rsync exclude syntax, so we can specify exclusions for each backup source.

## Installation

### Clone from github

```bash
git clone https://github.com/dusanx/saf
```
It is usually a good idea to put **saf** in path to be available on the command line, but it will work as good with specifying `/full/path/to/saf` on each invocation. Contibutors are welcome to create packages and send MR's for any specific OS or distribution.

### Packages for Arch or Debian or Ubuntu or Mint or Fedora or Gentoo or ...

Contributors are welcome.

## TL;DR usage

I want to create backup source point at my home folder, so I need to go there first:
```bash
cd ~
```
Initialize backup source and first local backup target (since we didn't specify --name, this target will be called B0):
```bash
saf init /mnt/backup_ssd/saf-backup/home
```
> **_NOTE:_**  Each **saf** command will require that backup.marker file exists at the destination and offer commands to create it if it is missing. This applies to any backup target, so **saf** can be sure that we can reach target location. In short just follow instructions when message appears.

Check and modify `.saf.conf` responsible for current folder:
```bash
saf status
saf configedit
```
Run the first backup (--name can specify target if there is more than one):
```bash
saf backup
```
Check the backups (--name can specify target if there is more than one):
```bash
saf list
saf realsizes
```
Add another target, this time over ssh on my-remote-server (specify target name too):
```bash
saf init --name my-vpn my-remote-server:/backup/saf-will-use-this/home
```
Add another target, this time on my Hetzner Storage Box (change u000000 with actual Hetzner user ID):
```bash
saf init --name H0 u000000@u000000.your-storagebox.de:/home/backup/saf-backup
```
> **_NOTE:_**  Make sure to check and edit `.saf.conf` to get all the parameters right, for instance remote storage box can be using different port than usual port 22.

With single backup source and three different backup targets, we can issue any command without specifying --name (meaning **saf** will use first target in the list, in our case `B0`) or by specifying any of the `B0`, `my-vpn`, or `H0` names:
```bash
saf list H0
saf realsizes my-vpn
saf backup my-vpn
saf rmrf H0 ./bin/zoom
saf backup # Note: this command will assume target B0 since it is the first target
saf freespace H0
```
> **_NOTE:_**  Every **saf** command is location relative. **saf** will find `.saf.conf` in current or any of the parent directories and perform command. First backup target is always considered default but all the commands can specify what target name they are referring to.

## Commands

### saf version [-h]

Print **saf** version.

### saf init [-h] [--name NAME] target

Initialize **saf** backup source at current folder and add first backup target.

### saf status [-h] [--all | --reverse]

Check status of the current directory, is it covered by any `.saf.conf` backup source.
Add --all to check all the locations up to root folder, since current folder can be covered by many different source locations.
Add --reverse to find all `.saf.conf` files recursively in all the sub-directories.
--all and --reverse can't be used at the same time, they are searching in different directions.

### saf configedit [-h] [--quiet | --verbose {0,1,2} | --debug]

Find and edit `.saf.conf` closest to current directory. This `.saf.conf` will be used for all the **saf** commands issued in current directory.

### saf list [-h] [--quiet | --verbose {0,1,2} | --debug] [name]

List all the backups in the specific [target-name] or using defult one. Sorted by time and date, with markers showing `keep-<period>` difference, as specified in `.saf.conf`

### saf realsizes [-h] [--quiet | --verbose {0,1,2} | --debug] [name]

Uses `du` to reach target location and check real folder sizes, since we are using rsync with hard links wherever possible.

### saf freespace [-h] [--quiet | --verbose {0,1,2} | --debug] [name]

Checks free space left on the specified or default target location.

### saf prune [-h] [--quiet | --verbose {0,1,2} | --debug] [name]

Prunes backups by criteria specifed in `.saf.conf`. Also executed automatically before each `saf backup`.

### saf backup [-h] [--quiet | --verbose {0,1,2} | --debug] [--resume] [name]

Runs the backup. It is probably a good idea to start it periodically for each backup target. Since **saf** is using current directory to determine what backup source and target to use, always cd to the desired directory before running backup. Example usage for the local user home directory, for the default backup target:
```bash
cd /home/user && saf backup
```

### saf rmrf [-h] [--quiet | --verbose {0,1,2} | --debug] [name] path

It is not unusual that some big or unwanted files or folders slip into backups. To prevent their future appearance we can edit `.saf.conf` to exclude such files or directories. However, as they remain present and taking space in the existing backups, `rmrf` offers elegant way to remove them. For instance, if my `H0` backup contains unwanted `~/bin/zoom`, I can easily remove it everywhere with
```bash
cd /home/user && saf rmrf H0 ./bin/zoom
```

> **_NOTE:_**  Using relative paths in `rmrf` is going to work since all the paths are calculated from backup source location.

### saf difference [-h] [--quiet | --verbose {0,1,2} | --debug] [name] backup

/* Experimental! */
This command should compare backup with the previous one and detect differences. Currently works well with the local file system backup targets but not with remote ones.

## Contributing

Contributors are welcome. Code, documentation or packaging improvements are welcome.

## Related projects

Thanks to [laurent22/rsync-time-backup](https://github.com/laurent22/rsync-time-backup) and [cytopia/linux-timemachine](https://github.com/cytopia/linux-timemachine) for the inspiration.

## License

**[MIT License](LICENSE.md)**

Copyright (c) 2015-2023 Dusan Popovic
