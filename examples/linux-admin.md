# Create New Users

1. Ssh into machine (port 4096, ip 192.120.0.1, user Laurel).
2. Change to root user. This is so sudo will not have to be used all the time.
3. Add financial group.
4. List what groups Greg belongs to.
5. Make Greg part of the financial group.
6. Create user Rajesh with sales being primary group. Make sure he is part of the financial and marketing group too. Make sure he has an expiration date of January 1st, 2024. Make sure he has a home directory.
7. Create user John under with admin group being Primary group. Have him replace his password every 4 weeks. Make his shell be zsh. Make sure he has a no home directory. Make sure he has UID 1200.
8. Add Stacy Gremoire with developper group being primary group. Make sure her shell is zsh. Save her full name under a comment. Make sure her home directory is titled gremoire. Copy over every file and directory under the /etc/skel/developper directory.
9. List all attached hard drives, usbs and etc. Make sure their filesystem types and part labels are listed
10. create /mnt directory.
11. Mount usb under sdb1 partition at the /mnt directory.
12. Move to /mnt directory.
13. Copy over calculate-profits.cpp and calculate-profits to Stacy's home directory.
14. Move to Stacy's home directory.
15. Unmount the usb.
16. Make sure Stacy is owner of calculate-profits.cpp and calculate-profits. Make sure sales group owns calculate-profits.
17. Make sure only Stacy has read and write permissions on calculate-profits.cpp.
18. Make sure only Stacy has read and write permissions on calculate-profits. Give execute permissions to group and user owner (sales and Stacy).
19. Make an exception for Greg with the calculate-profits file. Give him execute permissions (by using an ACL).
20. Check the resulting ACL.
21. Delete user John.
22. Create user Maurian with the same UID and other conditions that John had.
23. exit root.
24. exit ssh.

# Answers

```bash
ssh -p 4096 Laurel@192.120.0.1
su
groupadd financial
groups Greg
usermod -aG financial Greg
useradd -g sales -G financial,marketing -e 2024-01-01 -m Rajesh
useradd -g admin -f 28 -M -u 1200 -s $(which zsh) John
useradd -g developper -c 'Stacy Gremoire' -d /home/gremoire -k /etc/skel/developper -s $(which zsh) Stacy
lsblk -o +FSTYPE,PARTLABEL
mkdir /mnt
mount /dev/sdb1 /mnt
cd /mnt
cp calculate-profits.cpp /home/gremoire/
cp calculate-profits /home/gremoire/
cd /home/gremoire
umount -R /mnt
chown Stacy calculate-profits.cpp
chown Stacy:sales calculate-profits
chmod 700 calculate-profits.cpp
chmod u=rw,g=x,o-rwx calculate-profits
setfacl -m u:Greg:x calculate-profits
getfacl calculate-profits
userdel John
useradd -g admin -f 28 -M -u 1200 -o -s $(which zsh) Maurian
exit
```
