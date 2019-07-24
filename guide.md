# everyone-can-be-root
### Matthew Lister (chaosreader) & Ethan Payne (phat_sumo)

This is a guide to setting up a virtual machine hosting environment with VirtualBox and networking for hostname routing. 

We recommend installing on a machine with plenty of RAM & storage space- if you plan to have lots of VM's, you'll need the resources to match.

### Server installation

If you know how to install Ubuntu server, you can pick up at [environment setup]. TODO: link to below.

You'll probably need a bootable USB thumb drive to install Ubuntu Server on your chosen machine. Keep in mind that creating a bootable USB _will_ completely wipe the drive, so back up any important data before continuing.

Format your drive and remove any partitions.

Go ahead and download an [Ubuntu Server ISO](https://ubuntu.com/download/server), or simply `wget http://la-mirrors.evowise.com/ubuntu-releases/18.04/ubuntu-18.04.2-live-server-amd64.iso`


####  With terminal commands (Linux / OS X)

##### In Linux:

Plug in your USB, and identify it's device name.

```
lsblk
```

Will probably output something like:

```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 232.9G  0 disk
├─sda1   8:1    0   499M  0 part
├─sda2   8:2    0   100M  0 part /boot
├─sda3   8:3    0    16M  0 part
├─sda4   8:4    0 161.4G  0 part
└─sda5   8:5    0  70.9G  0 part /
sdb      8:16   0 931.5G  0 disk
├─sdb1   8:17   0 101.4G  0 part
├─sdb2   8:18   0    16G  0 part [SWAP]
└─sdb3   8:19   0 814.1G  0 part /home
sdc      8:32   1  14.9G  0 disk
```

In my case, my 16G drive shows up as `sdc`. When `sdX` is referenced later, just replace `X` with whatever letter corresponds to your thumb drive.

Now, copy your Ubuntu Server ISO onto the flash drive with `dd`. Be very careful to specify the correct drive- `dd` is known as 'disk destroyer' for a reason.

```
sudo dd if=ubuntu-18.04.2-live-server-amd64.iso of=/dev/sdX
```

If this method is causing problems, use an external program as shown below.

##### In Mac OS / OS X:


Plug in your USB, and identify it's device name.

```
diskutil list
```

Will probably output something like:

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *960.0 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk1         959.8 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +959.8 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            575.2 GB   disk1s1
   2:                APFS Volume Preboot                 45.0 MB    disk1s2
   3:                APFS Volume Recovery                510.3 MB   disk1s3
   4:                APFS Volume VM                      1.1 GB     disk1s4

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *16.0 GB    disk2
   1:        Apple_partition_map                         4.1 KB     disk2s1
   2:                  Apple_HFS                         2.5 MB     disk2s2
```

In my case, my 16G drive shows up as `disk2`. When `diskX` is referenced later, just replace `X` with whatever number corresponds to your thumb drive.

Now, copy your Ubuntu Server ISO onto the flash drive with `dd`. Be very careful to specify the correct drive- `dd` is known as 'disk destroyer' for a reason.

```
sudo dd if=ubuntu-18.04.2-live-server-amd64.iso of=/dev/diskX
```

If this method is causing problems, use an external program as shown below.

#### With external programs (all platforms)

Install a program like [Balena Etcher](https://www.balena.io/etcher/) or [Rufus](https://rufus.ie) (windows only). Follow the default instructions, specifying your USB drive and the Ubuntu Server ISO you downloaded earlier.

---

Plug your bootable USB into your machine, reboot, and [select your USB drive as the boot device.](https://www.howtogeek.com/129815/beginner-geek-how-to-change-the-boot-order-in-your-computers-bios/)

Once you're in, follow the on-screen instructions to install Ubuntu Server. Make sure to install OpenSSH Server, so we can remotely access the machine later.

### Environment Setup

Begin by installing some dependencies:

```
sudo apt update
sudo apt install virtualbox  virtualbox-guest-additions-iso
sudo apt install xorg
sudo apt install nginx
```

Download another copy of Ubuntu Server for later: 

```
wget http://la-mirrors.evowise.com/ubuntu-releases/18.04/ubuntu-18.04.2-live-server-amd64.iso
```

Start your X server and enable nginx to run on startup:

```
sudo systemctl start graphical.target
sudo sytemctl start nginx
sudo systemctl enable nginx
```

Now you can use `ssh` to remotely administer your machine, so you don't need to be in the same physical location. If you want to use a GUI program over ssh, you can use the `-X` argument:

```
ssh -X user@server
```

If you're having difficulty forwarding X windows, make sure `X11Forwarding yes` is set and uncommented in your `/etc/ssh/sshd_config`. 

### Create DNS Records

We need an A record to connect the name to the IP address. Then, we can add any number of CNAME records to point to our server's name. This can be emulated with the `/etc/hosts` file.

For our presentation, our hosts file looks like this:

TODO: add simplified hosts file for markdown

Notice how all those names resolve to the same IP address!

Next, we set up IP forwarding. Modify the `/etc/sysctl.conf` file by adding `net.ipv4.ip_forward = 1` to the end of the file. Then run:

```
sudo sysctl -p
```

This will allow IP forwarding when we later configure port forwarding via iptables to provide services other than http/https. 

### Set up VM's

Start virtualbox:

```
virtualbox
```

TODO: Include images

Create a new machine:

Allocate RAM. Remember, this allocation will be assigned whenever the virtual machine is turned on so it will come from the hypervisor’s total and not be available for other system processes. We chose 4gb per machine, but 2GB should be sufficient.

Create a virtual hard drive for the new machine. We chose VDI, 40gb, and dynamic allocation.

After the hard drive has been created, we will adjust the settings on our newly created VM. Remember to make all the configuration changes before clicking ‘ok’ or you will have to re-open the settings tab to finish.

Under the 'System' tab, you can increase the number of processors.

Under 'Storage', click on the CD icon. Check the Live CD/DVD box and click on the 2nd CD icon crop down menu. Select the iso that was downloaded earlier.

Under the network tab, Adapter 1 will show NAT. We want to enable adapter 2. Click the tab for adapter 2, select the check mark to “Enable network adapter”. Select “host-only adapter” from the drop down menu.  

Now we can click the start arrow on virtual box and install Ubuntu from the iso to our virtual machine’s hard drive. For example, we will install openssh during installation, and apache2 from the command line. We set up an apache server on this machine with the default values. We also created a second machine following the same steps as before.

Now we can set up nginx as proxy server. Nginx lives in /etc/nginx. The file /etc/nginx/sites-available/default is link to the sites-enabled folder so changes made to sites-available/default automatically makes things enabled. We will use nginx as a reverse proxy to mangle http packets sent to the physical machine’s ip address and forward them to the ip address of the virtual machine webservers. NOTE: These configurations work on a complete match principle. So while all virtual servers are listening for [anything]:80[port] the match comes from the server name. Multiple names can be given. Proxy_pass gives the internal ip of the virtual machines (from setting up the host-only adapter)

Next we will set up ip forwarding. Modify the /etc/sysctl.conf file by adding
net.ipv4.ip_forward = 1
to the end of the file. Then run 

```
sudo sysctl -p
```

On the new machine we will use our admin user to modify /etc/skel/. The contents of /etc/skel/  determine the initial contents of a users home directory.

> Tip: You can use `sudo su` to switch users and become 'root', the superuser. This removes the requirement to use `sudo` in priveleged directories. *Be cautious when executing commands as root, as they will not require a password to execute and may not provide warnings for their consequences.* Type `exit` or press Ctrl+d to return to your normal user.

```
sudo su 
cd /etc/skel
touch .Xauthority
mkdir Documents
mkdir Downloads
```





















