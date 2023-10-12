# BACKUPS IN GENERAL

Backups can be 1 of 3 types - full, incremental or differential. Full backup backups all files and folders. Incremental backup backups differences from last backup. Lastly, differential backup backups differences from last full backup.

# BACKUPS IN UNIX

Several different tools could be used to backup files on unix based systems (MAC, Linux, android - pretty much everything but Windows). This file will discuss 3 such tools: dd, rsync, and tar.

## dd

The dd command can really only be used to perform full backups. It is typically used to back up a disk partition or an entire drive. It can also can be used to copy a partition onto another partition. For example:

1. Change to root user
2. List all attached hard drives, usbs and etc. Make sure their filesystem types and part labels are listed
3. Mount the unmounted usb at /dev/sdb1 to mnt directory
4. Backup home directory (/dev/sda2) to a image file called backup.img on the mounted usb
5. Copy home partition over to second connected usb (/dev/sdc1).
6. Exit out of being root user

```bash
su
lsblk -o +FSTYPE,PARTLABEL
mount /dev/sdb1 /mnt
dd if=/dev/sda2 of=mnt/backup.img
dd if=/dev/sda2 of=/dev/sdc1
exit
```

## rsync

rsync is sort of a combination of full and incremental backup. In terms of the updating and backing up of files it is incremental, but in terms of the finished update it is a full backup. For example:

1. Change to root user
2. Backup the contents of /home/john to /backups/john
3. Backup the contents of /home/gracy to /backups/gracy. Make sure to exclude the directories titled classified and their contents. However, make an exception to that exclusion for any file or folder with its title beginnning with to-be-declassified.
4. Oops! Incorrect orders were given. Files and directories titled classified with their contents are to be backed up to /backups/classified. An exception to this are any file or folder with its title beginnning with to-be-declassified. These are to not be included in the backups/classified folder but instead be included within the backups/declassified folder. Good thing this interactive shell happens to be zsh. Delete all the classified and to-be-declassified files and folders that were backed up to /backups/gracy
5. Gracy is the chief manager of classified data in this organization. As such back up all the classified files of hers. Exclude the to-be-declassified content. Also it is the end of a backup cycle of sorts. Make sure classified content that is absent in her home directory is deleted in the backup location (/backups/classified)
6. Backup up all the to-be-declassified content under Gracy's home directory to /backups/declassified
7. Exit out of being root user

```bash
su
rsync -aPvz /home/john/ /backups/john
rsync -aPvz --include='to-be-declassified**' --exclude='classified/**' /home/gracy/ /backups/gracy
rm -rf /backups/gracy/{,**/}classified /backups/gracy/{,**/}to-be-declassified
rsync -aPvz --delete --include '**/' --include 'classified' --exclude 'to-be-declassified**' --include 'classified/**' --exclude '*' /home/gracy/ /backups/classified
rsync -avPz --include '**/' --include 'to-be-declassified**' --exclude '*' /home/gracy/ /backups/declassified
exit
```

## tar

tar command on unix can be used to do full, incremental and differential backups. Incremental and differential backups require the listed-incremental or g option with a meta file as the option's arguement. This option must be used on the full backup first though if one is to create incremental and differential backups. This option is also not needed if one is only doing full backups. Furthermore, a copy of the meta file created in the state is was when the full backup was made must be available. This is accomplished by copying the meta file. Examples:

1. Make a full backup
2. Some time has passed, make an incremental backup
3. You were only away from your desk for a little bit. In the same shell you made your previous incremental backup make another now you added another script file
4. Some time has passed, make a differential backup
5. Restore to the last incremental backup
6. Oops, the file version you wanted was in another incremental backup. Restore the first incremental backup
7. Some time has passed, restore to the only differential backup

For all the backups performed above they will:

1. be of .config/my-scripts backed up to files in backups directory
2. have a base file name starting with my-scripts
3. have a time stamp in base file name indicating year, month, day, hour, minute and second the full backup they are or are based on
4. have an indication of whether they are a full, incremental or differential backup in their base file name
5. have a time stamp in base file name indicating year, month, day, hour, minute and second they were made

Furthermore, the meta files of the backups shall have the same requirements above with the exception that only the differential backup meta files shall have a time stamp indicating when they were made. The shell being used is zsh. Lastly the backup up files must be gzip compressed and be created with the verbose or v flag of the tar command.

file: backup-helper-functions.sh
```bash
setup_backup_vars() {
	FULL_BACKUP_META_FILE="$FULL_BACKUP".snar
	INCR_BACKUP="$UNSUFFIXED_FULL_BACKUP"incr
	DIFF_BACKUP="$UNSUFFIXED_FULL_BACKUP"diff
	INCR_BACKUP_META_FILE="$INCR_BACKUP".snar
}
set_backup_locations() {
	[ $# -ge 2 ] || {
		echo "Error, need at least 2 arguements!" >&2
		return 1
	}
	BACKUP_RESTORE="$1"
	shift
	[ -d "$1" ] && {
		echo "Error, backup destination file $1 is a directory!" >&2
		return 2
}
	BACKUP_DIRECTORY=`dirname "$1"`
	BACKUP_BASE=`basename "$1"`
	[ $# -eq 1 ] && BACKUP_SOURCES=("$BACKUP_RESTORE") && return
	shift
	BACKUP_SOURCES=($@)
}
sorted_find() {
	[ $# -eq 2 ] || return 1
	find "$1" -type f -name "$2" -print0 | sort -z
}
find_newest() {
	[ $# -eq 2 ] || return 1
	sorted_find "$1" "$2" | tail -zn 1
}
find_newest_full_backup() {
	FULL_BACKUP_FILE=`find_newest "$BACKUP_DIRECTORY" "$BACKUP_BASE".'????_??_??-??:??:??'full.tar.gz` || return 1
	FULL_BACKUP_FILE=`basename "$FULL_BACKUP_FILE"`
	FULL_BACKUP="${FULL_BACKUP_FILE%.tar.gz}"
	UNSUFFIXED_FULL_BACKUP="${FULL_BACKUP%full}"
	FULL_BACKUP_DATE="${UNSUFFIXED_FULL_BACKUP##*.}"
	setup_backup_vars
}
get_date() {
	date '+%Y_%m_%d-%H:%M:%S'
}
make_full_backup() {
	FULL_BACKUP_DATE="$(get_date)"
	UNSUFFIXED_FULL_BACKUP="$BACKUP_BASE"."$FULL_BACKUP_DATE"
	FULL_BACKUP="$UNSUFFIXED_FULL_BACKUP"full
	FULL_BACKUP_FILE="$FULL_BACKUP".tar.gz
	setup_backup_vars
	tar -cvzf "$BACKUP_DIRECTORY/$FULL_BACKUP_FILE" -g "$BACKUP_DIRECTORY/$FULL_BACKUP_META_FILE" $BACKUP_SOURCES || {
		echo "Error, either you don't have permissions to access the file or the parent directory $BACKUP_DIRECTORY does not exist!" >&2
		return 1
	}
	cp "$BACKUP_DIRECTORY/$FULL_BACKUP_META_FILE" "$BACKUP_DIRECTORY/$INCR_BACKUP_META_FILE"
}
make_incr_backup() {
	tar -cvzf "$BACKUP_DIRECTORY/$INCR_BACKUP$(get_date).tar.gz" -g "$BACKUP_DIRECTORY/$INCR_BACKUP_META_FILE" $BACKUP_SOURCES || {
		echo "Error, either you don't have permissions to access the file or the parent directory $BACKUP_DIRECTORY does not exist!" >&2
		return 1
	}
}
make_diff_backup() {
	local DIFF_DATE="$(get_date)"
	local DIFF_META_FILE="$DIFF_BACKUP$DIFF_DATE".snar
	cp "$BACKUP_DIRECTORY/$FULL_BACKUP_META_FILE" "$BACKUP_DIRECTORY/$DIFF_META_FILE"
	tar -cvzf "$BACKUP_DIRECTORY/$DIFF_BACKUP$DIFF_DATE.tar.gz" -g "$BACKUP_DIRECTORY/$DIFF_META_FILE" $BACKUP_SOURCES || {
		echo "Error, either you don't have permissions to access the file or the parent directory $BACKUP_DIRECTORY does not exist!" >&2
		return 1
	}
}
is_a_num() {
	[[ "$1" =~ '^[1-9][0-9]*$' ]] || {
		echo "Error, arguement needs to be a nonzero number!" >&2
		return 1
	}
}
restore_full_backup() {
	tar -xvzf "$BACKUP_DIRECTORY/$FULL_BACKUP_FILE" -g /dev/null "$BACKUP_RESTORE"
}
restore_inc_backup() {
	is_a_num "$1" || return 1
	restore_full_backup
	local INCR_BACKUP_FILES=`sorted_find "$BACKUP_DIRECTORY" "$INCR_BACKUP"'????_??_??-??:??:??'.tar.gz`
	local i=1
	while [ $i -le $1 ]
	do
		local INCR_BACKUP_FILE=`sed -zn "$i"p <<< "$INCR_BACKUP_FILES"`
		[ "$INCR_BACKUP_FILE" ] || break
		tar -xvzf "$INCR_BACKUP_FILE" -g /dev/null "$BACKUP_RESTORE"
		((i++))
	done
	[ $i -eq 1 ] || return 0
	echo "No incremental backups based on $FULL_BACKUP_FILE present. Only did full backup"
}
restore_diff_backup() {
	is_a_num "$1" || return 1
	restore_full_backup
	local DIFF_BACKUP_FILES=`sorted_find "$BACKUP_DIRECTORY" "$DIFF_BACKUP"'????_??_??-??:??:??'.tar.gz`
	local DIFF_BACKUP_FILE=`sed -zn "$1"p <<< "$DIFF_BACKUP_FILES"`
	[ "$DIFF_BACKUP_FILE" ] || {
		local DIFF_PLURALITY='backup'
		[ $1 -gt 1 ] && DIFF_PLURALITY="$DIFF_PLURALITY"s
		echo "There is no differential backup based on $FULL_BACKUP_FILE $1 $DIFF_PLURALITY away!"
		echo "WHat $DIFF_BACKUP_FILES $INCR_BACKUP_FILES"
		return 1
	}
	tar -xvzf "$DIFF_BACKUP_FILE" -g /dev/null "$BACKUP_RESTORE"
}
```

```bash
source backup-helper-functions.sh
set_backup_locations backups/my-scripts .config/my-scripts
make_full_backup
```

```bash
source backup-helper-functions.sh
set_backup_locations backups/my-scripts .config/my-scripts
find_newest_full_backup
make_incr_backup
make_incr_backup
```

```bash
source backup-helper-functions.sh
set_backup_locations backups/my-scripts .config/my-scripts
find_newest_full_backup
make_diff_backup
```

```bash
source backup-helper-functions.sh
set_backup_locations backups/my-scripts .config/my-scripts
find_newest_full_backup
restore_inc_backup 9
restore_inc_backup 1
```

```bash
source backup-helper-functions.sh
set_backup_locations backups/my-scripts .config/my-scripts
find_newest_full_backup
restore_diff_backup 1
```

The above was tested and works on the latest arch linux version and zsh version 5.9. This will likely differ in functionality between Unix OSes since the commands in the above examples lack some options in on Unix OS and command version while also posssessing different flags the other(s) don't have. For example, the z option of sort and tail command is lacking in Mac OS 
