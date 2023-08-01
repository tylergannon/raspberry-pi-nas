# Raspberry Pi NAS / Timemachine

Use these instructions:

https://ovechkin.xyz/blog/2021-12-13-using-raspberry-pi-for-time-machine

(Instructions from [jeremycollins.net](https://www.jeremycollins.net/using-a-raspberry-pi-as-a-nas-mac-os-time-machine) seem good but the other actually works.)

## Initial Setup

Install the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and run it to write your base image.  Be sure to use the settings wizard to set
a username and auth key.  The way I did it was to check "Allow public-key authentication only".  Then inside of `set authorized_keys ... `, copy / paste
the contents of the `my_key.pub` for the SSH key that will be used to access
this Raspberry Pi.

You have to also select the `Set username and password` option, to enter the
desired username.  Before doing this, I was unable to log in to my machine.

Configure the wlan settings and you're good to go.

Now just `ssh` into the rberry pi machine using the hostname as configured in the imager program.

### Share Definitions

lines 167: from `/etc/samba/smb.conf`

```
#======================= Share Definitions =======================

[timemachine]
    comment = Backups
    path = /mnt/timemachine/backups
    valid users = macdorfenburger
    read only = no
    vfs objects = catia fruit streams_xattr
    fruit:time machine = yes


[media]
    comment = Media
    path = /mnt/media
    valid users = macdorfenburger
    read only = no
    vfs objects = catia fruit streams_xattr
    fruit:time machine = yes

```

### /etc/avahi/services/samba.service
```
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">%h</name>

  <!-- Add service for "timemachine" share -->
  <service>
    <type>_smb._tcp</type>
    <port>445</port>
    <txt-record>sys=waMa=0</txt-record>
    <txt-record>model=Xserve1,1</txt-record>
    <txt-record>dk0=adVN=timemachine,adVF=0x82</txt-record>
    <txt-record>adVF=0x100</txt-record>
  </service>

  <!-- Add service for "media" share -->
  <service>
    <type>_smb._tcp</type>
    <port>445</port>
    <txt-record>sys=waMa=0</txt-record>
    <txt-record>model=Xserve1,1</txt-record>
    <txt-record>dk0=adVN=media,adVF=0x82</txt-record>
    <txt-record>adVF=0x100</txt-record>
  </service>

  <!-- Add device-info service (optional) -->
  <service>
    <type>_device-info._tcp</type>
    <port>9</port>
    <txt-record>model=Xserve1,1</txt-record>
  </service>
</service-group>
```
