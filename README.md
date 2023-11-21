# Running a file sytem via systemd-nspawn
This is a quick tutorial for those who want to know how to run a **systemd** and **dbus** inside a file sytem. This guide will demonstrate how to install file system of *Ubuntu-23.10-Mantic-amd64* via *systemd-nspawn*.

## The main difference between systemd-nspawn and Docker
At the moment, the *systemd-nspawn* could run **systemd** and **dbus** inside the file system in a regular way, but the Docker **can't**. It means the file system via *systemd-nspawn* could be treated as a real system. 

## Install systemd-nspawn on the host system
*user@host:/#* `sudo apt-get install systemd-container`

## Bootstrap a basic file system of *Ubuntu-23.10-mantic_amd64* to */var/lib/machines/Ubuntu-23.10-mantic-amd64*
This step probably requires the time of a cup of coffee to finish.

*user@host:/#* `sudo debootstrap --include=systemd,dbus --verbose --arch amd64 Ubuntu-23.10-mantic /var/lib/machines/Ubuntu-23.10-mantic http://archive.ubuntu.com/ubuntu`

After the fetching and depacking processes are done, the *debootstrap* will provide a minimally bootable file system.

## Change the root password and add a default user before logging into the file system of *Ubuntu-23.10-mantic-amd64*
Because the fresh file system of *Ubuntu-23.10-mantic-amd64* doesn't have a password for root and also does not exist for the default user. Thus, we have to setup a password for root and create a default user for the file system of *Ubuntu-23.10-mantic-amd64*. 

### Use `mount` to mount the file system of *Ubuntu-23.10-mantic-amd64*

*user@host:/#*
```bash
  #!/bin/bash
  CONTAINER="/var/lib/machines/Ubuntu-23.10-mantic-amd64"
  sudo mount -t proc none $CONTAINER/proc 2> /dev/null
  sudo mount -t sysfs sys $CONTAINER/sys  2> /dev/null 

  sudo mount -o bind /dev $CONTAINER/dev 2> /dev/null
  sudo mount -o bind /dev/pts $CONTAINER/dev/pts 2> /dev/null
  sudo mount -o bind /dev/tty $CONTAINER/dev/tty 2> /dev/null

  sudo mount -o bind /opt $CONTAINER/opt 2> /dev/null
  sudo mount -o bind /home $CONTAINER/home 2> /dev/null
  sudo mount -o bind /root $CONTAINER/root 2> /dev/null
```

### Use *chroot* to switch to the file system of *Ubuntu-23.10-mantic-amd64*
*user@host:/#* ``sudo chroot /var/lib/machines/Ubuntu-23.10-mantic-amd64 /bin/bash``

There is a different prompt: *root@hpc:/#* shows in the terminal if chroot is successful. In the meantime, we could use the system tools in the file system of *Ubuntu-23.10-mantic-amd64* to setup the password for root and create a default user.

#### Give a password for root
Now you have been switched into the file system of *Ubuntu-23.10-mantic-amd64* via *chroot*. So here just enter the *passwd* to setup password for root.

*root@container:/#* ```passwd```

#### Create a default user and give him a password
And also create a default user via command of *useradd*.

*root@container:/#*
```bash
  #/bin/bash
  BOOTSTRAP_DEFAULT_USER=default_user_name
  useradd -m -d /home/${BOOTSTRAP_DEFAULT_USER} -s /bin/bash ${BOOTSTRAP_DEFAULT_USER}
  passwd ${BOOTSTRAP_DEFAULT_USER}
```

#### Quit chroot and unmount all associated devices
At the moment, the file system had a password for root and a default user. So we don't need to stay here for long. We leave the file system of *Ubuntu-23.10-mantic-amd64* via *quit* immediately.

*root@container:/#* ``quit``

Clear it up by unmount all associated devices. Please keep in mind that you should reboot the host computer if the umount is failed. *(The umount is often successful. But somehow, the devices can't umount successful because they are busy.)*.

*user@host:/#* 
```bash
  #/bin/bash
  CONTAINER="/var/lib/machines/Ubuntu-23.10-mantic-amd64"

  sudo mount -t proc none $CONTAINER/proc 2> /dev/null
  sudo mount -t sysfs sys $CONTAINER/sys  2> /dev/null 

  sudo mount -o bind /dev $CONTAINER/dev 2> /dev/null
  sudo mount -o bind /dev/pts $CONTAINER/dev/pts 2> /dev/null
  sudo mount -o bind /dev/tty $CONTAINER/dev/tty 2> /dev/null

  sudo mount -o bind /opt $CONTAINER/opt 2> /dev/null
  sudo mount -o bind /home $CONTAINER/home 2> /dev/null
  sudo mount -o bind /root $CONTAINER/root 2> /dev/null
```

## Boot up the container of *Ubuntu-23.10-mantic_amd64* via Host's *systemd*
There has been created a *root file system* and the *default user* we needed. So we just boot the container of `Ubuntu-23.10-mantic-amd64` up via *systemd-nspawn*. The *--bind /opt:/opt* is useful parameter for mapping devices between *host* and *container*.

*user@host:/#* ```sudo systemd-nspawn --boot --machine=Ubuntu-23.10-mantic-amd64 --bind /opt:/opt --private-users=off```

The following message will show up if it succeeds.

```bash
Spawning container mantic-amd64 on /opt/rootfs/mantic-amd64.
Press Ctrl-] three times within 1s to kill container.
systemd 253.5-1ubuntu6 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT -GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY +P11KIT +QRENCODE +TPM2 +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -BPF_FRAMEWORK -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified)
Detected virtualization systemd-nspawn.
Detected architecture x86-64.

Welcome to Ubuntu 23.10!

Hostname set to <hpc>.
Queued start job for default target graphical.target.
[  OK  ] Created slice system-modprobe.slice - Slice /system/modprobe.
[  OK  ] Created slice user.slice - User and Session Slice.
[  OK  ] Started systemd-ask-password-console.path - Dispatch Password Requests to Console Directory Watch.
[  OK  ] Started systemd-ask-password-wall.path - Forward Password Requests to Wall Directory Watch.
[  OK  ] Reached target cryptsetup.target - Local Encrypted Volumes.
[  OK  ] Reached target integritysetup.target - Local Integrity Protected Volumes.
[  OK  ] Reached target paths.target - Path Units.
[  OK  ] Reached target slices.target - Slice Units.
[  OK  ] Reached target swap.target - Swaps.
[  OK  ] Reached target veritysetup.target - Local Verity Protected Volumes.
rpcbind.socket: Failed to create listening socket (0.0.0.0:111): Address already in use
rpcbind.socket: Failed to listen on sockets: Address already in use
rpcbind.socket: Failed with result 'resources'.
[FAILED] Failed to listen on rpcbind.socket - RPCbind Server Activation Socket.
See 'systemctl status rpcbind.socket' for details.
[DEPEND] Dependency failed for rpcbind.service - RPC bind portmap service.
rpcbind.service: Job rpcbind.service/start failed with result 'dependency'.
[DEPEND] Dependency failed for rpc-statd.service - NFS status monitor for NFSv2/3 locking..
rpc-statd.service: Job rpc-statd.service/start failed with result 'dependency'.
[  OK  ] Reached target rpcbind.target - RPC Port Mapper.
[  OK  ] Listening on syslog.socket - Syslog Socket.
[  OK  ] Listening on systemd-initctl.socket - initctl Compatibility Named Pipe.
[  OK  ] Listening on systemd-journald-dev-log.socket - Journal Socket (/dev/log).
[  OK  ] Listening on systemd-journald.socket - Journal Socket.
         Mounting dev-hugepages.mount - Huge Pages File System...
         Mounting proc-fs-nfsd.mount - NFSD configuration filesystem...
         Starting systemd-journald.service - Journal Service...
         Starting keyboard-setup.service - Set the console keyboard layout...
         Mounting sys-fs-fuse-connections.mount - FUSE Control File System...
         Starting systemd-remount-fs.service - Remount Root and Kernel File Systems...
[  OK  ] Finished systemd-remount-fs.service - Remount Root and Kernel File Systems.
         Starting systemd-sysusers.service - Create System Users...
[  OK  ] Started systemd-journald.service - Journal Service.
         Starting systemd-journal-flush.service - Flush Journal to Persistent Storage...
[  OK  ] Mounted dev-hugepages.mount - Huge Pages File System.
[  OK  ] Mounted proc-fs-nfsd.mount - NFSD configuration filesystem.
[  OK  ] Mounted sys-fs-fuse-connections.mount - FUSE Control File System.
[  OK  ] Finished systemd-sysusers.service - Create System Users.
         Starting systemd-tmpfiles-setup-dev.service - Create Static Device Nodes in /dev...
[  OK  ] Finished systemd-tmpfiles-setup-dev.service - Create Static Device Nodes in /dev.
[  OK  ] Finished keyboard-setup.service - Set the console keyboard layout.
[  OK  ] Reached target local-fs-pre.target - Preparation for Local File Systems.
[  OK  ] Reached target local-fs.target - Local File Systems.
         Starting console-setup.service - Set console font and keymap...
[  OK  ] Finished console-setup.service - Set console font and keymap.
[  OK  ] Finished systemd-journal-flush.service - Flush Journal to Persistent Storage.
         Starting systemd-tmpfiles-setup.service - Create Volatile Files and Directories...
[  OK  ] Finished systemd-tmpfiles-setup.service - Create Volatile Files and Directories.
         Mounting run-rpc_pipefs.mount - RPC Pipe File System...
         Starting systemd-resolved.service - Network Name Resolution...
[  OK  ] Reached target time-set.target - System Time Set.
         Starting systemd-update-utmp.service - Record System Boot/Shutdown in UTMP...
[  OK  ] Mounted run-rpc_pipefs.mount - RPC Pipe File System.
[  OK  ] Reached target rpc_pipefs.target.
         Starting nfs-blkmap.service - pNFS block layout mapping daemon...
         Starting nfsdcld.service - NFSv4 Client Tracking Daemon...
[  OK  ] Reached target nfs-client.target - NFS client services.
[  OK  ] Reached target remote-fs-pre.target - Preparation for Remote File Systems.
[  OK  ] Reached target remote-fs.target - Remote File Systems.
[  OK  ] Finished systemd-update-utmp.service - Record System Boot/Shutdown in UTMP.
[  OK  ] Started nfsdcld.service - NFSv4 Client Tracking Daemon.
[  OK  ] Started nfs-blkmap.service - pNFS block layout mapping daemon.
[  OK  ] Started systemd-resolved.service - Network Name Resolution.
[  OK  ] Reached target network.target - Network.
[  OK  ] Reached target network-online.target - Network is Online.
[  OK  ] Reached target nss-lookup.target - Host and Network Name Lookups.
[  OK  ] Reached target sysinit.target - System Initialization.
[  OK  ] Started apt-daily.timer - Daily apt download activities.
[  OK  ] Started apt-daily-upgrade.timer - Daily apt upgrade and clean activities.
[  OK  ] Started dpkg-db-backup.timer - Daily dpkg database backup timer.
[  OK  ] Started e2scrub_all.timer - Periodic ext4 Online Metadata Check for All Filesystems.
[  OK  ] Started logrotate.timer - Daily rotation of log files.
[  OK  ] Started motd-news.timer - Message of the Day.
[  OK  ] Started systemd-tmpfiles-clean.timer - Daily Cleanup of Temporary Directories.
[  OK  ] Reached target timers.target - Timer Units.
[  OK  ] Listening on avahi-daemon.socket - Avahi mDNS/DNS-SD Stack Activation Socket.
[  OK  ] Reached target sockets.target - Socket Units.
[  OK  ] Listening on dbus.socket - D-Bus System Message Bus Socket.
         Starting nfs-idmapd.service - NFSv4 ID-name mapping service...
[  OK  ] Reached target basic.target - Basic System.
         Starting avahi-daemon.service - Avahi mDNS/DNS-SD Stack...
[  OK  ] Started cron.service - Regular background program processing daemon.
         Starting dbus.service - D-Bus System Message Bus...
[  OK  ] Started dmesg.service - Save initial kernel messages after boot.
[  OK  ] Started fsidd.service - NFS FSID Daemon.
         Starting nfs-mountd.service - NFS Mount Daemon...
         Starting nmbd.service - Samba NMB Daemon...
         Starting rsyslog.service - System Logging Service...
         Starting samba-ad-dc.service - Samba AD Daemon...
         Starting systemd-logind.service - User Login Management...
         Starting systemd-user-sessions.service - Permit User Sessions...
[  OK  ] Finished systemd-user-sessions.service - Permit User Sessions.
[  OK  ] Started console-getty.service - Console Getty.
[  OK  ] Created slice system-getty.slice - Slice /system/getty.
[  OK  ] Reached target getty.target - Login Prompts.
[  OK  ] Started nfs-idmapd.service - NFSv4 ID-name mapping service.
[  OK  ] Started systemd-logind.service - User Login Management.
[  OK  ] Started rsyslog.service - System Logging Service.
[  OK  ] Started nfs-mountd.service - NFS Mount Daemon.
         Starting nfs-server.service - NFS server and services...
[  OK  ] Started dbus.service - D-Bus System Message Bus.
[  OK  ] Started avahi-daemon.service - Avahi mDNS/DNS-SD Stack.
[  OK  ] Finished nfs-server.service - NFS server and services.
         Starting rpc-statd-notify.service - Notify NFS peers of a restart...
[  OK  ] Started rpc-statd-notify.service - Notify NFS peers of a restart.
[  OK  ] Started nmbd.service - Samba NMB Daemon.
         Starting smbd.service - Samba SMB Daemon...
[  OK  ] Started smbd.service - Samba SMB Daemon.
[  OK  ] Reached target multi-user.target - Multi-User System.
[  OK  ] Reached target graphical.target - Graphical Interface.
         Starting systemd-update-utmp-runlevel.service - Record Runlevel Change in UTMP...
[  OK  ] Finished systemd-update-utmp-runlevel.service - Record Runlevel Change in UTMP.

Ubuntu 23.10 hpc pts/0

hpc login: 
```

Ok, there is a normal login prompt here. It does mean everything is fine and doing well. So let's congratulate it!


