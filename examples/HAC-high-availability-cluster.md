# HAC (High Availability Cluster)

A computer cluster is essentially many seperate computers acting as one.
This could be for redundancy (backups), fault tolerance (ensuring more than one physical device can serve some resources so if one stops working one of the others serves those resources), load sharing (performance) or any combination of the 3.
Clusters are supercomputers.
Generally some Linux distro is used for this.
High Availability cluster is one that is focused on fault tolerance.
This document describes an nginx cluster of 2 computer with a 3rd VM being used as an ISCSI server for shared storage of the website files.
This document uses Fedora server for the OS within a VirtualBox VM (virtual machine).
Furthermore this example uses btrfs for the file system and raid 10.
Btrfs is used because it is free and supports both shrinking and growing.
This was pushed out to github latter than desired since I kept running into tech I would have to pay for when building This example within a VM.
I would then have to research some free equivalent.

# Setting Up the Virtual Network

**1.** Click on the **3 bars icon** within **Tools**.

**2.** Select **Nat Networks**.

**3.** Click **Create**

**4.** Name it **NatNetworkForPacemaker**.

**5.** Choose the appropriate value for **IPv4 Prefix** and note it for latter.
The format of the value will be *choosen range starting IP*/*choosen cidr net mask*.
All the following commands will have **192.168.100.0** as the *choosen range starting IP* and *24* as the *choosen cidr net mask*.

**6.** Click **Apply**.

**7.** Click **fedora server**.

**8.** Click **Settings**.

**9.** Click **Network**.

**10.** Make sure **Enable Network Adapter** is checked.

**11.** For **Attached to** select **NAT Network**.

**12.** For **Name** select **NatNetworkForPacemaker**.

**13.** Under **Promiscuous Mode** select **Allow All**.

**14.** Click **Ok**.

# Creating Virtual Disks for the Logical Volume


**15.** Click **Machine**, **New** and then give the **Name** of **nginx physical volume 1**.

**16.** Make sure the **ISO Image** is **\<not selected\>**.

**16.** Give it the **Type** of **Linux**.

**17.** Click **Next**.

**18.** Click **Next**.

**19.** Give it the **Disk Size** of **20 GB**.

**20.** Click **Next**.

**21.** Click **Finish**.

**22.** Right click **nginx physical volume 1**.

**23.** Click **Clone**.

**24.** Give it the **Name** of **nginx physical volume 2**.

**25.** Click **Next**.

**26.** Click **Finish**.

**27.** Give it the **Name** of **nginx physical volume 3**.

**28.** Click **Next**.

**29.** Click **Finish**.

**30.** Give it the **Name** of **nginx physical volume 4**.

**31.** Click **Next**.

**32.** Click **Finish**.

# Setting Up the Fedora Server


**33.** Go to [https://fedoraproject.org/en/server/download](https://fedoraproject.org/en/server/download) and download the latest Fedora server DVD iso for your archetecture.

**34.** Open VirtualBox.

**35.** Click **Machine**, **New** and then give the name of **fedora server**.

**36.** For **ISO image** supply the location that you downloaded the Fedora Server above.

**37.** Click **Next**.

Most of the rest of the following instructions under this section (Fedora Server Installation) will differ according to one's hardware.


**38.** Make sure the **Base Memory**, **Processors** and **Enable EFI (special OSes only)** are set appropriately for your hardware.

**39.** Click **Next**.

**40.** Under **Create a Virtual Hard Disk Now** set the appropriate amount of space.

**41.** Click **Next**.

**42.** Click **Finish**.

**43.** Now click on the **fedora server** machine and then click **Settings**.

**44.** Click **Network**.

**45.** Give it the **Attached to** of **NAT Network**.

**46.** Make sure the **Name** is **NatNetworkForPacemaker**.

**47.** Click **Ok**.

# Fedora Server Installation


**48.** Now click on the **fedora server** machine and then click **Start**.

**49.** Wait for it to boot up and the select **Install Fedora** *(Your version here)*.

**50.** Select your language and then select **Continue**.

**51.** Select **Root Account**.

**52.** Click **Enable root account** and then set your chosen password under both **Root password** and **Confirm**.

**53.** Click **Allow SSH login with password**.

**54.** Click **Done**.

**55.** You can then select **Installation Destination** and set up the hard disk for **Automatic** or **Custom**.

**56.** Click **Done** and then **Begin Installation**.

# Setting Up of the Fedora Servers


**57.** Enter **root** for user name and the *password you chose above (Fedora Server Installation)*.

**58.** Run
```sh
vim /etc/hosts
```
.

**59.** Type **Go** and then add the following:
> choose-an-ip pacemaker-main
> choose-an-ip pacemaker-clone1
> choose-an-ip pacemaker-iscsi
where each of the *choose-an-ip*s will be unique and within the *choosen range starting IP*.
The following steps use **192.168.100.12**, **192.168.100.13** and **192.168.100.14** for the *choose-an-ip* of pacemaker-main, pacemaker-clone1 and pacemaker-iscsi.
So the actual lines that should be added if this example is followed is
> 192.168.100.12 pacemaker-main
> 192.168.100.13 pacemaker-clone1
> 192.168.100.14 pacemaker-iscsi
.
If however you have different ip network and/or choosen IPs these will differ.

**60.** Press the **escape key**.

**61.** Type **:wq**.

**62.** Press the **enter key** to exit vim.

**63.** Run
```sh
firewall-cmd --permanent --add-port=3260/tcp
```
.

# Making a Clone of the Server for ISCSI


**64.** Right click the same **fedora server** VM and hover over **Stop**.

**65.** Click **Power off**.

**66.** Wait for it to power off.

**67.** Right click the same **fedora server** VM and select **Clone**.

**68.** Give it the name of **nginx iscsi target**.

**69.** Click **Next**.

**70.** Click **Finish**.

**71.** Click **nginx iscsi target**.

**72.** Click **Settings**.

**73.** Click **Network**.

**74.** Under **MAC Address** click the refresh button.

**75.** Click **Storage**.

**76.** Click **Adds a hard disk** icon to the far right of **Controller: SATA**.

**77.** Select the **nginx physical volume 1** disk.

**78.** Click **Choose**.

**79.** Click **Adds a hard disk** icon to the far right of **Controller: SATA**.

**80.** Select the **nginx physical volume 2** disk.

**81.** Click **Choose**.

**82.** Click **Adds a hard disk** icon to the far right of **Controller: SATA**.

**83.** Select the **nginx physical volume 3** disk.

**84.** Click **Choose**.

**85.** Click **Adds a hard disk** icon to the far right of **Controller: SATA**.

**86.** Select the **nginx physical volume 4** disk.

**87.** Click **Choose**.

**88.** Click **Ok**.

# Setting Up of the Pacemaker Nodes


**89.** Click **fedora server**.

**90.** Click **Start**.

**91.** Log in on the **fedora server**.

**92.** Run
```sh
yum install pcs pacemaker nginx corosync iscsi*
```
.

**93.** Run
```sh
ls -lZ /var/www
```
and take note that selinux type context of the html directory is **httpd_sys_content_t**.

**94.** Run
```sh
mkdir /nginx-site
```
.

**95.** Run
```sh
semanage fcontext -a -t httpd_sys_content_t '/nginx-site/site(/.*)?'

**96.** Run
```sh
firewall-cmd --permanent --add-service=http
```
.

**97.** Run
```sh
firewall-cmd --permanent --zone=public --add-service=http
```
.

**98.** Run
```sh
firewall-cmd --permanent --add-service=https
```
.

**99.** Run
```sh
firewall-cmd --permanent --zone=public --add-service=https
```
.

**100.** Run
```sh
semanage boolean --modify --on httpd_can_network_relay
```
.

**101.** Run
```sh
semanage boolean --modify --on httpd_can_network_connect
```
.

**102.** Run
```sh
vim /etc/nginx/nginx.conf
```
.

**103.** Add the following nested indented lines to to the http server block:
> location / {
> 	root /nginx-site/site;
> }
.

**104.** Save and exit vim.

**105.** Run
```sh
vim /usr/bin/mount-nginx-volume
```
.

**106.** Add the lines:
```sh
#!/usr/bin/bash
vgchange -a y nginx_vg
mount /dev/nginx_vg/nginx_lv /nginx-site
```
.

**107.** Save and exit vim.

**108.** Run
```sh
chmod +x /usr/bin/mount-nginx-volume
```
.

**109.** Run
```sh
vim /etc/systemd/system/mount-nginx-volume.service
```
.

**110.** Add the lines:
```toml
[Unit]
Description=Activate ISCSI volume group and mount nginx logical volume
Requires=iscsid.service
After=iscsid.service

[Service]
ExecStart=/usr/bin/mount-nginx-volume
Type=simple
```
.

**111.** Save and exit vim.

**112.** Run
```sh
passwd hacluster
```
and set your choosen password.

**113.** On both **pacemaker-main** and **pacemaker-clone1** run
```sh
systemctl enable --now pcsd
systemctl property set stonith-enabled=false
systemctl property set no-quorum-policy=ignore
```

# Making a Clone of the Server for the Cluster


**114.** Right click the same **fedora server** VM and hover over **Stop**.

**115.** Click **Power off**.

**116.** Wait for it to power off.

**117.** Right click the same **fedora server** VM and select **Clone**.

**118.** Give it the name of **fedora server clone 1**.

**119.** Click **Next**.

**120.** Click **Finish**.

**121.** Click **fedora server clone 1**.

**122.** Click **Settings**.

**123.** Click **Network**.

**124.** Under **MAC Address** click the refresh button.

**125.** Click **Ok**.

# Setting up Static IPs

For **fedora server**, **fedora server clone 1** and **nginx iscsi target** perform the following:


**126.** Click on the server.

**127.** Click **Start**.

**128.** Select the first **Fedora Linux** *...version information* that doesn't have the word rescue in it.

**129.** Enter **root** for user name and *password you chose above (Fedora Server Installation)* for the password.

**130.** Run
```sh
hostnamectl set-hostname appropriate-hostname
```
.
For **fedora server** *appropriate-hostname* will be **pacemaker-main**.
For **fedora server clone 1** *appropriate-hostname* will be **pacemaker-clone1**.
For **nginx iscsi target** *appropriate-hostname* will be **pacemaker-iscsi**.

**131.** Run
```sh
ip addr
```
 to get the network interface name of your virtual ethernet interface name.
 The ethernet name will begin with en and will be at the start of 1 of the lines lacking indention.
 It will occur immmediately after the starting number and colon.

**132.** Run
```sh
nmcli con mod your-network-interface ipv4.addresses choose-an-ip/choose-a-cidr-netmask
```
.
Where *your-network-interface* is the interface gotten from the step directly above, *choose-an-ip* is an IP within your *choosen range starting IP* of step 5 and *choose-a-cidr-netmask* is *choosen cidr net mask* of step 5.
Also *choose-an-ip* must be uniqe for **fedora server**, **fedora server clone 1** and **nginx iscsi target**.
All the following commands will use **24** as the *choose-a-cidr-netmask*.
All the following commands will use **192.168.100.12**, **192.168.100.13** and **192.168.100.14** for the *choose-an-ip* of **pacemaker-main**, **pacemaker-clone1** and **pacemaker-iscsi** respectively as defined in step **59**.
Otherwise step **59** should have different lines for addding to the **/etc/hosts** file that correspond to each of the IPs choosen (each *choose-an-ip*).

**133.** Run
```sh
nmcli con mod your-network-interface ipv4.gateway choose-an-ip-plus-one
```
.
Where *choose-an-ip-plus-one* is *choosen range starting IP* of step 5 with 1 add to it (in a arithmetic sense).

**134.** Run
```sh
nmcli con mod your-network-interface ipv4.dns dns-server-ip
```
.
Where *dns-server-ip* is the IP of your prefered or needed DNS.
A DNS (Domain Name Server) is a server responsable for translating domains, basically human understandable website names (like www.google.com), into IP addresses that the computer understands.

**135.** Run
```sh
nmcli con mod your-network-interface ipv4.method manual
```
.

**136.** Finally run
```sh
nmcli con up your-network-interface
```
.

# Setting Up the Raid 10 Logical Volume for Nginx


**137.** Click **nginx iscsi target**.

**138.** Click **Start**.

**139.** Log in on the **nginx iscsi target**.

**140.** Run
```sh
lsblk -o +FSTYPE,LABEL
```
to get locations of the **nginx physical volume 1** through **4**.
They should be in the form of **sd**_x_, where x is a letter.
These should be **sdb**, **sdc**, **sdd** and **sde**.

**141.** For each of the **nginx physical volumes** format them as a physical volume (in volume management terms).
The location of a **nginx physical volume** should be **/dev/**_location of nginx physical volume in step above_.
Thus, assuming the locations are **sdb**, **sdc**, **sdd** and **sde** run:
```sh
pvcreate /dev/sdb
pvcreate /dev/sdc
pvcreate /dev/sdd
pvcreate /dev/sde
```
.
If the location for any of the **nginx physical volumes** differ, like say instead of **sdb** it is **sdh** replace the corresponding command using **/dev/sd**_x_ with the appropriate location (in this case
```sh
pvcreate /dev/sdh
pvcreate /dev/sdc
pvcreate /dev/sdd
pvcreate /dev/sde
```
).
The same is said of any of the command below that use **/dev/sd**_x_.
The commands/examples below will all use **sdb**, **sdc**, **sdd** and **sde**.

**142.** Run
```sh
vgcreate --setautoactivation n nginx_vg /dev/sdb /dev/sdc /dev/sdd /dev/sde
```
.

**143.** Run
```sh
lvcreate -l 25%FREE -n nginx_lv nginx_vg
```
.

**144.** Run
```sh
vgchange -a y nginx_vg
```
.

**145.** Run
```sh
mkfs.btrfs /dev/nginx_vg/nginx_lv
```
.

**146.** Run
```sh
mount /dev/nginx_vg/nginx_lv /mnt
```
.

**147.** Run
```sh
mkdir /mnt/site
```
.

**148.** Run
```sh
vim /mnt/site/index.html
```
.

**149.** Add the following lines:
> \<html>
> \<head>
> \</head>
> \<body>
> 	\<p>This is running with pacemaker.\</p>
> \</body>
> \</html>
.

**150.** Save and exit vim.

**151.** Run
```sh
chcon -t httpd_sys_content_t /mnt/site /mnt/site/index.html
```
.

**152.** Run
```sh
umount /mnt
```
.

**153.** Run
```sh
vgchange -a n nginx_vg
```
.

# Setting Up ISCSI Initiators/Clients


**154.** Start both **pacemaker-main** and **pacemaker-clone1**.

**155.** In both **pacemaker-main** and **pacemaker-clone1** run
```sh
systemctl enable --now iscsid
```
.

**156.** In both **pacemaker-main** and **pacemaker-clone1** run
```sh
cat /etc/iscsi/initiatorname.iscsi
```
and it will output InitiatorName=*iqn*.
Note the *iqn* of each of the 2 as it is needed latter

# Setting Up the Rest of ISCSI in Its Entirety


**157.** Switch to **pacemaker-iscsi**.

**158.** Run
```sh
yum install targetcli
```
.

**159.** Run
```sh
systemctl enable --now target
```
.

**160.** Run
```sh
targetcli
```
.

**161.** Run
```sh
backstores/block create vol1 /dev/sdb
backstores/block create vol2 /dev/sdc
backstores/block create vol3 /dev/sdd
backstores/block create vol4 /dev/sde
iscsi create
ls
```
.
Note the resulting iqn listed under iscsi.

**162.** Run
```sh
iscsi/iqn/tpg1/acls create iqn
```
twice (1 for **pacemaker-main** and 1 for **pacemaker-clone1**) where *iqn* is the *iqn* of step **155**.

**163.** Run
```sh
iscsi/iqn/tpg1/acls/iqn create 0 /backstores/block/vol1
iscsi/iqn/tpg1/acls/iqn create 1 /backstores/block/vol2
iscsi/iqn/tpg1/acls/iqn create 2 /backstores/block/vol3
iscsi/iqn/tpg1/acls/iqn create 3 /backstores/block/vol4
```
twice (1 for **pacemaker-main** and 1 for **pacemaker-clone1**) where *iqn* is the *iqn* of step **155**.
Don't bother running the **iscsi/iqn/tpg1/acls/**_iqn_ **create 0 /backstores/block/vol**_n_ for the ones that are already autocreated (sometimes some are).

**164.** Run
```sh
saveconfig
exit
```
.

**165.** Keep **pacemaker-iscsi** running and switch to both **pacemaker-main** and **pacemaker-clone1**.

**166.** On both **pacemaker-main** and **pacemaker-clone1** run
```sh
iscsiadm -m node -T iqn -p 192.168.100.14 -l
```
where *iqn* is the *iqn* of step **160**.

# Setting Up Pacemaker


**167.** Switch to either **pacemaker-main** or **pacemaker-clone1**.

**168.** Run
```sh
pcs host auth pacemaker-main pacemaker-clone1
```

**169.** Enter hacluster for the user name and the *password choosen in step 112* for the password.

**170.** Run
```sh
pcs cluster setup nginx pacemaker-main pacemaker-clone1
pcs cluster start --all
pcs resource create mount_nginx systemd:mount-nginx-volume
pcs resource create nginx_vip ocf:heartbeat:IPaddr2 ip=192.168.100.15 cidr_netmask=24
pcs resource create nginx systemd:nginx
pcs constraint order stop mount_nginx then nginx
pcs constraint order nginx_vip then nginx
```
.

**171.** Running
```sh
curl 192.168.100.15
```
from either **pacemaker-iscsi**, **pacemaker-main** or **pacemaker-clone1** should give
> \<html>
> \<head>
> \</head>
> \<body>
> 	\<p>This is running with pacemaker.\</p>
> \</body>
> \</html>
.

**172.** Placing either of the pacemaker nodes on standby but not both should have the services restart on the other nodes.
