# Running a container via systemd-nspawn
This is a tutorial for those who want to know how to run a systemd inside a container.

## The main different between systemd-nspawn container and Docker container
At the moment, the container via systemd-nspawn could run a **systemd**, **dbus** inside in a regular way, but the Docker container **can't**. It means the container via systemd-nspawn could be treated as a real system. This guide will provide a basic installation guide to install `Ubuntu-23.10-Mantic-amd64` via systemd-nspawn.

## Install systemd-nspawn on the host system
`sudo apt-get install systemd-container`

## Bootstrap a basic Ubuntu-23.10-mantic_amd64 in /var/lib/machines/mantic-amd64
This step probably requires the time of a cup of coffee.

`sudo debootstrap --include=systemd,dbus --verbose --arch amd64 mantic /var/lib/machines/mantic-amd64 http://archive.ubuntu.com/ubuntu`

After the fetching and depacking processes are done, the debootstrap will provide a minimally bootable system.

## Change the root password and add a default user before logging into the container of Ubuntu-23.10-mantic_amd64
Because the fresh container of `Ubuntu-23.10-mantic_amd64` doesn't have a password for root and also does not exist for the default user. Thus, we have to setup a password for root and create a default user for the container of `Ubuntu-23.10-mantic_amd64`. 

### Use `mount` to mount the container
```bash
  ROOT="/var/lib/machines/mantic-amd64"
  sudo mount -t proc none $ROOT/proc 2> /dev/null
  sudo mount -t sysfs sys $ROOT/sys  2> /dev/null 

  sudo mount -o bind /dev $ROOT/dev 2> /dev/null
  sudo mount -o bind /dev/pts $ROOT/dev/pts 2> /dev/null
  sudo mount -o bind /dev/tty $ROOT/dev/tty 2> /dev/null

  sudo mount -o bind /opt $ROOT/opt 2> /dev/null
  sudo mount -o bind /home $ROOT/home 2> /dev/null
  sudo mount -o bind /root $ROOT/root 2> /dev/null
```

### Use `chroot` to switch to the container
```sudo chroot /var/lib/machines/mantic-amd64 /bin/bash```

There is a different prompt: `root@hpc:/#` shows in the terminal if chroot is successful. In the meantime, we could use the system tool in the container of `Ubuntu-23.10-mantic_amd64` to update the password for root and create a default user.

#### Change password for root
```passwd```

#### Create a default user




