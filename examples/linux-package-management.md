# Searching for Packages

## Arch Based

```bash
pacman -Ss package
```

## Debian Based

```bash
apt-get search package
```
matches on package name whereas
```bash
apt-cache search package
```
or
```bash
apt search package
```
matches on package name and long description.

## Gentoo Based

```bash
emerge --search package
```
matches on name whereas
```bash
emerge --searchdesc package
```
matches on name and description and
```bash
emerge --s regex
```
does the same but uses regex expression.


## OpenSUSE Based

```bash
zypper search package
```
or
```bash
zypper se package
```

## Red Hat Based

```bash
yum search package
```
generally for older versions and
```bash
dnf search package
```
for newer versions

# Installing Packages

## Arch Based

```bash
pacman -S package
```

## Debian Based

```bash
apt-get install package
```
or
```bash
apt install package
```
matches on package name and long description.

## Gentoo Based

```bash
emerge package
```

## OpenSUSE Based

```bash
zypper install package
```
or
```bash
zypper in package
```

## Red Hat Based

```bash
yum install package
```
generally for older versions and
```bash
dnf install package
```
for newer versions

# Removing Packages

## Arch Based

```bash
pacman -Rns package
```
-n option insures that no file associated with the package that would otherwise be backed up will be backed up. -s option insure that dependecies that are not needed elsewhere are deleted as well.

## Debian Based

```bash
apt-get remove package
```
or
```bash
apt remove package
```
to remove the just package.
```bash
apt-get purge package
```
or
```bash
apt purge package
```
to remove configuration files too.
```bash
apt-get autoremove
```
or
```bash
apt autoremove
```
to remove dependencies that are no longer needed.

## Gentoo Based

```bash
emerge -c package
```
or
```bash
emerge --depclean package
```
deletes package and its dependecies that are not needed elsewhere.
Probably should use --ask and --verbose options to make sure that packages that are are wanted to be kept are not deleted.
```bash
emerge -W package
```
or
```bash
emerge --deselect package
```
Removes just the package and keeps all of its dependecies.
```bash
emerge -c
```
or
```bash
emerge --depclean
```
Removes orphaned dependecies.
Notice that these 2 commands are the same as the commands used to remove a package with the exception of package being left out.
Probably should use --ask and --verbose options.

## OpenSUSE Based

```bash
zypper remove package
```
or
```bash
zypper rm package
```

## Red Hat Based

```bash
yum remove package
```
or
```bash
yum erase package
```
generally for older versions and
```bash
dnf remove package
```
for newer versions.
```bash
yum autoremove package
```
generally for older versions and
```bash
dnf autoremove package
```
for newer versions will remove package and all its no longer needed dependecies.
```bash
yum autoremove
```
generally for older versions and
```bash
dnf autoremove
```
for newer versions will remove all unneeded dependecies.

# Updating Packages

## Arch Based

```bash
pacman -Sy
```

## Debian Based

```bash
apt-get update
```
or
```bash
apt update
```

## Gentoo Based

```bash
emerge --sync
```

## OpenSUSE Based

```bash
zypper refresh
```
or
```bash
zypper ref
```

## Red Hat Based

```bash
yum check-update
```
generally for older versions and
```bash
dnf check-update
```
for newer versions

# Upgrading Packages

## Arch Based

```bash
pacman -Su
```
but I usually run
```bash
pacman -Syu
```
to update and upgrade in one go.

## Debian Based

```bash
apt-get upgrade
```
or
```bash
apt upgrade
```

## Gentoo Based

```bash
emerge -uD @world
```

## OpenSUSE Based

```bash
zypper update
```
or
```bash
zypper up
```

## Red Hat Based

```bash
yum upgrade
```
generally for older versions and
```bash
dnf upgrade
```
for newer versions
