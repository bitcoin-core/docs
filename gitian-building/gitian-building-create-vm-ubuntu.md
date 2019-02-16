# Setup Ubuntu virtual machine on VirtualBox

Table of Contents
------------------

- [Create a new VirtualBox VM](#create-a-new-virtualbox-vm)
- [Connecting to the VM](#connecting-to-the-vm)

Create a new VirtualBox VM
---------------------------
In the VirtualBox GUI click "New" and choose the following parameters in the wizard:

![](figs/create_new_vm_ubuntu.png)

- Type: Linux, Ubuntu (64-bit)

![](figs/create_vm_memsize.png)

- Memory Size: at least 3000MB, anything less and the build might not complete.

![](figs/create_vm_hard_disk.png)

- Hard Disk: Create a virtual hard disk now

![](figs/create_vm_hard_disk_file_type.png)

- Hard Disk file type: Use the default, VDI (VirtualBox Disk Image)

![](figs/create_vm_storage_physical_hard_disk.png)

- Storage on physical hard disk: Dynamically Allocated

![](figs/create_vm_file_location_size.png)

- File location and size: at least 40GB
- Click `Create`

After creating the VM, we need to configure it.

- Click the `Settings` button, then go to `System` tab and `Processor` sub-tab. Increase the number of processors to the number of cores on your machine if you want builds to be faster.

![](figs/system_settings.png)

- Go to the `Network` tab. Adapter 1 should be attached to `NAT`.

![](figs/network_settings.png)

- Click `Advanced`, then `Port Forwarding`. We want to set up a port through which we can reach the VM to get files in and out.
- Create a new rule by clicking the plus icon.

![](figs/port_forwarding_rules.png)

- Set up the new rule the following way:
  - Name: `SSH`
  - Protocol: `TCP`
  - Leave Host IP empty
  - Host Port: `22222`
  - Leave Guest IP empty
  - Guest Port: `22`

- Click `Ok` twice to save.

Get the [Ubuntu 18.04.x minimal iso image](http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/mini.iso) (a more recent minor version should also work, see also [Installation/MinimalCD](https://help.ubuntu.com/community/Installation/MinimalCD)).
This DVD image can be [validated](https://help.ubuntu.com/community/Installation/MinimalCD) using a SHA1 hashing tool, for example on
Unixy OSes by entering the following in a terminal:

    echo "cce936c1f9d1448c7d8f74b76b66f42eb4f93d4a  mini.iso" | sha1sum -c
    # (must return OK)

Replace `sha1sum` with `shasum` on OSX.

Then start the VM. On the first launch you will be asked for a CD or DVD image. Choose the downloaded ISO.

![](figs/select_startup_disk_ubuntu.png)

Installing Ubuntu
------------------

This section will explain how to install Ubuntu on the newly created VM.

- Choose the non-graphical installer (only if you will use the standard iso image instead of the suggested mini.iso, you will see the graphical installer choice).  We do not need the graphical environment; it will only increase installation time and disk usage.

![](figs/ubuntu_install_1_boot_menu.png)

**Note**: Navigating in the Ubuntu installer:
To keep a setting at the default and proceed, just press `Enter`.
To select a different button, press `Tab`.

- Choose locale and keyboard settings (doesn't matter, you can just go with the defaults or select your own information)

![](figs/ubuntu_install_2_select_a_language.png)
![](figs/ubuntu_install_3_select_location.png)
![](figs/ubuntu_install_4_configure_keyboard1.png)
![](figs/ubuntu_install_4_configure_keyboard2.png)

- The VM will detect network settings using DHCP, this should all proceed automatically
- Set the hostname `ubuntu`.
<!--
  - Leave domain name empty.
-->
![](figs/ubuntu_install_5_network1_hostname.png)

  - Choose a mirror (any will do)

![](figs/ubuntu_install_5_network2_mirror_country.png)
![](figs/ubuntu_install_5_network3_mirror.png)

  - Enter proxy information (unless you are on an intranet, leave this empty)

![](figs/ubuntu_install_5_network4_proxy.png)
<!--
![](figs/ubuntu_install_6_domain_name.png)

- You can leave the root password empty. Otherwise, enter it twice and remember it for later.

![](figs/ubuntu_install_6a_set_up_root_password.png)
-->

- Wait a bit while 'Loading additional components' runs
- Name the new user `gitianuser` (the full name doesn't matter, you can leave it empty)
- Set the account username as `gitianuser`

![](figs/ubuntu_install_7_set_up_user_fullname.png)
![](figs/ubuntu_install_8_set_up_username.png)

- Choose a user password and enter it twice (remember it for later)

![](figs/ubuntu_install_9_user_password.png)

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

- The base system will be installed, this will take some minutes
- Select how to manage system upgrades -> *No automatic updates*

![](figs/ubuntu_install_17_software_updates.png)

- Wait a bit while 'Select and install software' runs
- Choose software to install. We need just the base system.
- Make sure only 'OpenSSH server' and 'Basic Ubuntu Server' are checked

![](figs/ubuntu_install_19_software_selection.png)

- Install the GRUB boot loader to the master boot record? -> Yes

![](figs/ubuntu_install_20_install_grub.png)

- Is the system clock set to UTC? -> *Yes*

![](figs/ubuntu_install_21_configure_clock.png)

- Installation Complete -> *Continue*
- After installation, the VM will reboot and you will have a working Ubuntu VM. Congratulations!

![](figs/ubuntu_install_22_finish_installation.png)

Connecting to the VM
----------------------

After the VM has booted you can connect to it using SSH, and files can be copied from and to the VM using a SFTP utility.
Connect to `localhost`, port `22222` (or the port configured when installing the VM).
On Windows you can use [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and [WinSCP](http://winscp.net/eng/index.php).

For example, to connect as `gitianuser` from a Linux command prompt use

      $ ssh gitianuser@localhost -p 22222
      The authenticity of host '[localhost]:22222 ([127.0.0.1]:22222)' can't be established.
      RSA key fingerprint is ae:f5:c8:9f:17:c6:c7:1b:c2:1b:12:31:1d:bb:d0:c7.
      Are you sure you want to continue connecting (yes/no)? yes
      Warning: Permanently added '[localhost]:22222' (RSA) to the list of known hosts.
      gitianuser@localhost's password: (enter gitianuser password configured during install)

      System information as of Sat Sep 22 19:33:49 EDT 2018

      System load:  0.0               Processes:             97
      Usage of /:   8.8% of 39.12GB   Users logged in:       0
      Memory usage: 4%                IP address for enp0s3: 10.0.2.15
      Swap usage:   0%

    0 packages can be updated.
    0 updates are security updates.



    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.

    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

    gitianuser@ubuntu:~$

Optional - Easier login to the VM
---------------------------------

You'll need to generate an SSH key, e.g. by following the instructions under "Generating a new SSH key" [here](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

After that, login to the VM and enter:

```bash
mkdir .ssh
```

On your machine edit or create `~/.ssh/config` and add:

```bash
Host gitian
    HostName localhost
    Port 22222
    User gitianuser
```

Open a new terminal tab and enter:

```bash
scp ~/.ssh/id_rsa.pub gitian:.ssh/authorized_keys
```

Next time you need to login to the VM, just use: `ssh gitian`
