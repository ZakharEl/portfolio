# Log Files Location

Logs are located in the /var/log/ directory.
A centralized log is often found in /etc/syslog or /etc/syslogd.

# Examples

1. change to root user to avoid typing sudo all the time.
2. check the info concerning the booting of the device.
3. check the boot number -23.
4. Check authentication attempts.
5. List all attached hard drives, usbs and etc. Make sure their filesystem types and part labels are listed
6. create /mnt directory.
7. Mount usb under sdb1 partition at the /mnt directory.
8. Check the journal of the connected device since 1 weak ago. Make sure it is at the end of the file.
9. Unmount the usb.
10. check kernel messages.
11. exit root

```bash
su
journalctl --list-boots
journalctl -b -23
cat /var/log/auth.log
lsblk -o +FSTYPE,PARTLABEL
mkdir /mnt
mount /dev/sdb1 /mnt
journalctl -D /mnt/var/log/journal -e -S '1 week'
umount -R /mnt
dmesg
exit
```
