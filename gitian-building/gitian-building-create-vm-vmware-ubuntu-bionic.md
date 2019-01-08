Table of contents
-----------------
- [Setup Ubuntu virtual machine on VMWare Player](#setup-ubuntu-virtual-machine-on-vmware-player)
  - [Create a new VMWare Player VM](#create-a-new-vmware-player-vm)
      - [Get the Ubuntu 18.04 net installer](#get-the-ubuntu-1804-net-installer)
    - [Configure VM](#configure-vm)
  - [Installing Ubuntu](#installing-ubuntu)
  - [Connecting to the VM](#connecting-to-the-vm)
  - [Optional - Easier login to the VM with public key](#optional---easier-login-to-the-vm-with-public-key)

Setup Ubuntu virtual machine on VMWare Player
=============================================

Create a new VMWare Player VM
-----------------------------

First we have to download latest Ubuntu net install ISO.

#### Get the Ubuntu 18.04 net installer
Get the [Ubuntu 18.04 LTS (Bionic Beaver) net installer](http://archive.ubuntu.com/ubuntu/dists/bionic-updates/main/installer-amd64/current/images/netboot/mini.iso) (a more recent minor version should also work, see also [Ubuntu Network installation](http://cdimage.ubuntu.com/netboot/)).
This ISO/DVD image can be validated using a SHA256 hashing tool, for example on
Unixy OSes by entering the following in a terminal:

    echo "2a1416fa51448feff9e1e7bc399a7354d546a94044c58ab08f9bb387c3afcbae  mini.iso" | sha256sum -c
    # (must return OK)

Replace `sha256sum` with `shasum` on OSX.


In the VMWare Player GUI click "Create a New Virtual Machine" and choose the following parameters in the wizard:

![](figs/create_new_vm_vmware_ubuntu.png)

- Install operating system from ISO image

![](figs/create_vm_vmware_select_guest_os_ubuntu.png)

- Type: Linux, Ubuntu 18.04 Bionic (64-bit)

![](figs/create_vm_vmware_machine_name.png)

- Set machine name.
  _We will use following name for this guide:_ `gitianbuild`

![](figs/create_vm_vmware_disk_size.png)

- Maximum disk size: at least 40GB
- _We will store disk as single file in this guide, you can split it too_
- Click "Next"

### Configure VM

To [configure VM](#configure-vm) click on `Customize Hardware`

![](figs/create_vm_vmware_customize_hw_ubuntu.png)


![](figs/create_vm_vmware_memsize.png)

- Memory Size: at least 3000MB, anything less and the build might not complete.


![](figs/system_settings_vmware_cpu.png)

- Increase the number of processors to the number of cores on your machine if you want builds to be faster.

![](figs/create_vm_vmware_finish_configure_ubuntu.png)

- Check settings once again and click `Finish`

_Note_: _Marking checkbox `Automatically power on this virtual machine after creation will start the VM after clicking finish and closing next window:_

![](figs/create_vm_vmware_created.png)

Installing Ubuntu
------------------

This section will explain how to install Ubuntu on the newly created VM.

- Choose the non-graphical installer.  We do not need the graphical environment; it will only increase installation time and disk usage.

![](figs/ubuntu_install_1_boot_menu.png)

**Note**: Navigating in the Ubuntu installer:
To keep a setting at the default and proceed, just press `Enter`.
To select a different button, press `Tab`.

- Choose locale and keyboard settings (doesn't matter, you can just go with the defaults or select your own information)

![](figs/ubuntu_install_2_select_a_language.png)
![](figs/ubuntu_install_3_select_location.png)
![](figs/ubuntu_install_4_configure_keyboard.png)
![](figs/ubuntu_install_4_2_configure_keyboard.png)
![](figs/ubuntu_install_4_3_configure_keyboard.png)

- The VM will detect network settings using DHCP, this should all proceed automatically
- Configure the network:
  - Hostname `ubuntu`.
  - Leave domain name empty.
- Configure the mirror
  - _any suggested can be used, closest is in most cases faster_
- Proxy, if not used, leave it empty

![](figs/ubuntu_install_5_configure_the_network.png)

- Choose a mirror (any will do)

![](figs/ubuntu_install_5_2_configure_mirror.png)
![](figs/ubuntu_install_5_3_configure_mirror.png)

- Enter proxy information (unless you are on an intranet, leave this empty)

![](figs/ubuntu_install_5_4_configure_proxy.png)

- Name the new user `gitianuser` (the full name doesn't matter, you can leave it empty)
- Set the account username as `gitianuser`

![](figs/ubuntu_install_7_set_up_user_fullname.png)
![](figs/ubuntu_install_8_set_up_username.png)

- Choose a user password, enter and verify your password (__remember it for later__)

![](figs/ubuntu_install_9_user_password.png)
![](figs/ubuntu_install_9_1_user_password.png)

- The installer will set up the clock using a time server; this process should be automatic
- Set up the clock: choose a time zone (depends on the locale settings that you picked earlier; specifics don't matter)  

![](figs/ubuntu_install_10_configure_clock.png)

- Disk setup
  - Partitioning method: Guided - Use the entire disk

![](figs/ubuntu_install_11_partition_disks.png)

  - Select disk to partition: SCSI1 (0,0,0)

![](figs/ubuntu_install_12_choose_disk.png)

  - Finish partitioning and write changes to disk -> *Yes* (`Tab`, `Enter` to select the `Yes` button)

![](figs/ubuntu_install_15_write_changes.png)

- Manage updates 

![](figs/ubuntu_install_18_pam_configuration.png)

- The base system will be installed, this will take a minute or so

![](figs/ubuntu_install_19_software_selection.png)

- Install the GRUB boot loader to the master boot record? -> Yes

![](figs/ubuntu_install_20_install_grub.png)

- Device for boot loader installation -> ata-VBOX_HARDDISK

![](figs/ubuntu_install_21_install_grub_bootloader.png)

- Installation Complete -> *Continue*
- After installation, the VM will reboot and you will have a working Ubuntu VM. Congratulations!

![](figs/ubuntu_install_22_finish_installation.png)

Connecting to the VM
----------------------

After the VM has booted you can connect to it using SSH, and files can be copied from and to the VM using a SFTP utility.

1. Find out your IP address:
   Login to your VM and run `ifconfig` to find IP address of your vmware network adapter:
   ```bash
   ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.123.143  netmask 255.255.255.0  broadcast 172.16.123.255
        inet6 fe80::20c:29ff:fe7c:a908  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7c:a9:08  txqueuelen 1000  (Ethernet)
        RX packets 62147  bytes 79269431 (79.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17692  bytes 25055408 (25.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

   ```
2. On your machine edit or create `~/.ssh/config` and add:
   ```bash
   host gitian
       HostName 172.16.123.143
       Port 22
       PreferredAuthentications password
       User gitianuser
   ```
3. Connect to `gitian`.
  _On Windows you can use [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and [WinSCP](http://winscp.net/eng/index.php)._

  For example, to connect as `gitianuser` from a Linux command prompt use
  
      $ ssh gitian
      The authenticity of host '[gitian]:22 ([127.0.0.1]:22)' can't be established.
      RSA key fingerprint is ae:f5:c8:9f:17:c6:c7:1b:c2:1b:12:31:1d:bb:d0:c7.
      Are you sure you want to continue connecting (yes/no)? yes
      Warning: Permanently added '[gitian]:22' (RSA) to the list of known hosts.
      gitianuser@gitian's password: (enter gitianuser password configured during install)

      The programs included with the Ubuntu GNU/Linux system are free software;
      the exact distribution terms for each program are described in the
      individual files in /usr/share/doc/*/copyright.
  
      Ubuntu GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
      permitted by applicable law.
      gitianuser@ubuntu:~$

Use `sudo` to execute commands as root.

Optional - Easier login to the VM with public key
-------------------------------------------------

For easier login with public key you'll need to generate an SSH key, e.g. by following the instructions under "[Generating a new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)".

After that, login to the VM and enter:

```bash
mkdir .ssh
```

On your machine edit or create `~/.ssh/config` and add:

```bash
host gitian
    HostName 172.16.123.143
    Port 22
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
    User gitianuser
```

Open a new terminal tab and enter:

```bash
scp ~/.ssh/id_rsa.pub gitian:.ssh/authorized_keys
```

Next time you need to login to the VM, just use: `ssh gitian`