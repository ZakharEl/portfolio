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
		echo "There is no differential backup based on $FULL_BACKUP_FILE $1 $DIFF_PLURALITY away!" >&2
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

The above was tested and works on the latest arch Linux version and zsh version 5.9. This will likely differ in functionality between Unix OSes since the commands in the above examples lack some options in on Unix OS and command version while also posssessing different flags the other(s) don't have. For example, the z option of sort and tail command is lacking in Mac OS

# Backups in Windows

Several approaches exist for backup in Windows as well.

## dd

Although the script shown in the **dd** subsection of the **BACKUPS IN UNIX** above can be run only on unix platforms it can be used to copy an entire hard drive that has Windows on it! Just make sure to not have both the old and new hard drive running at the same time. Also problems might arrive if the hard drive you are copying from is bigger than the one you are restoring to. Any ISO disk images of Windows are not bootable but they can be used to restore to a hard the system in full. That hard drive would then boot.

## FILE HISTORY

Open **settings** in Windows. Click on the **Updates and Security**. In left pane click on **Backup**. Connect external hard drive or ssd to computer. Click on **Add a drive** under the main section titled **Backup**. To include filed and folders right under where **Add a drive** was click more options. Then click on **Add a folder** under the **back up these folders**. Finally select what folder or file you want backed up. The file and folders to exclude and time interval for the backups can also be changed under **Backup options**. To resore files and folders with file history one can restore it from the **file explorer** or **Control Panel**.

To restore files and folders with file history in the **file explorer** have the file of folder location opened up in **file explorer** and click it once. Then click the **home** tab of the ribbon at the top and click on **history** in the resulting drop down panel. Click on left and right arrows to view the file as it was backwards and forwards in time of available backups. Click the green center button between the left and right arrows to confirm this as the version you want. Finally double click **Replace the file in the destination**.

To restore files and folders with file history in the **Control Panel** open **Control Panel**. Click on **File History**. In the left pane click **Restore personal files**. Click on left and right arrows to view the available backups backwards and forward in time. Then click the file you want and click the the green center button between the left and right arrows to confirm this as the version you want. Finally double click **Replace the file in the destination**.

## BACKUP AND RESTORE

**Backup and Restore** is used to create a system image like **dd** described aboved. It is a good option to backup the an entire Windows installation for restoring the installation in its entirety.

To perform the backup part of Backup and Restore open **Settings**. Click **Update and Security**. Click **Backup** in the left panel. Click **Go to Backup and Restore (Windows 7)**. Click **Create a system image** in the left panel. Select the desired location to backup the system (in the form of a system image) and then click **Next**. Finally make sure everything is correct and click **Start backup**.

One can perform the restore part of Backup and Restore in a number of ways. One is to select the **Repair your computer option** after booting from a Windows installation ISO. Another is from within **Advanced startup**. To get to **Advanced startup** open **Settings**. Click **Go to Backup and Restore (Windows 7)**. Next click **Restart now** under **Advance startup**. From within **Advanced startup** select **Troubleshoot**. Next click **Advanced options**. Click **System Image Recovery** (**See more recovery options**). Select your user account and then enter your passsword. Make sure the information presented in the **Select a system image backup** is correct. If not enter the correct information. Next click **Select a system image**. click **Next**. Select the appropriate system image location and then click **Next**. After that select the correct date and time of the backup. Make sure you don't need any additional options in this step for things like (re)formatting the drive of installation of any drivers. Perform the additional options if needed then click **Next**. Click **Finish**. Finally Click **Yes**.

## GROUP POLICY OBJECT

Using a **GPO** (group policy object) is a good option when backing up files to multiple computer within an AD (Active Directory) Domain and only 1 or a handful of administrators will alter or add to the backup files in question. An example would to automate installation of some programs to many computer. Everyone must have the appropriate permissions for this to work. Open **Group Policy Management**. Right click **Group Policy Objects** and select **New**. Give it an appropriate name and then click **Ok**. Right click the policy object that was just created and click **Edit**. Expand **Computer Configuration**, **Preferences**, and finally **Windows settings**. Right click **Files**. Click **New** and then **File**. Under **Source file(s):** enter the network share that the computers to restore to have access to. Under **Destination File:** enter the location on each of the computers you would like to restore it to. Click **Ok**. Close the policy object. In **Group Policy Management** right click the OU (Organizational Unit) of the computers you wish to restore to. Select **Link an Existing GPO**. Finally select the GPO created earlier and click **Ok**. Either wait for the GPO to automatically take effect (typically 90 minutes) or run GPupdate /force on the command line.

## OTHER METHODS

Some other methods exist for creating backups for Windows systems. Largely ones using powershell or bash scripts with some cli tools. In Windows cli tools like robocopy, xcopy and Copy-Item could be used with Get-Date for timestamps and gzip for backup compressions. ActiveDirectory module would also be best to use. adtool on Linux could be used to interact with active director in Linux for Windows systems. If I explained more of these in detail I might actually hurt my chances at getting a job.
