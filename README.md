## How do we protect our configurations

We use LUKS (Linux Unified Key Setup) and [cryptsetup](https://gitlab.com/cryptsetup/cryptsetup) is the solution we are using.

## Disclaimer
This solution is for linux only but similar concept can be applied to other platforms.

All setup/access of luks requires root, but the process owner is a normal user.

####Dependencies (Install them if not already installed)

```
# cryptsetup
yum install cryptsetup-luks

# losetup
```

####First Time Setup

Create a [loop device](https://en.wikipedia.org/wiki/Loop_device)

```
# create an empty file (padded with zeros)
# 10MB should be more than enough for configuration storage
dd if=/dev/zero of=pass.iso bs=1M count=10

# create loop device
losetup --find --show pass.iso

# list out the devices
losetup -a
```

Encrypt the partition (this step will reset all data on this partition)

```
# enter a strong passphrase when requested
cryptsetup -y -v luksFormat /dev/loop0

# note: The passphrase cannot be recovered if lost, so be careful.

cryptsetup luksOpen /dev/loop0 enc

# create a filesystem (it will format)
mkfs.ext4 /dev/mapper/enc

mkdir /opt/pass
mount /dev/mapper/enc /opt/pass

# create configuration files
cd /opt/pass

# for Ruby on Rails
# vi secrets.yml
# vi database.yml

# give proper access rights
# chown user:group *.yml
```

Create symlink to point yml to the drive

```
# for Ruby on Rails
# cd config
# ln -s /opt/pass/secrets.yml
# ln -s /opt/pass/database.yml
```

Close everything

```
cd ..

umount /opt/pass
cryptsetup luksClose enc
```

#### Startup Process
Open and Mount

```
# use the passphrase from previous step when requested
# interactive mode
cryptsetup luksOpen /dev/loop0 enc

# non-interactive mode (when starting the process remotely)
# echo -n '<password>' | cryptsetup luksOpen /dev/loop0 enc

mount /dev/mapper/enc /opt/pass
```

Start your service

```
# e.g. service xxx start
# the configs are usually loaded onto memory by now.
```

Unmount and Close

```
umount /opt/pass
cryptsetup luksClose enc
```

## Reference

[Cryptsetup Guide](http://www.cyberciti.biz/hardware/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/)
