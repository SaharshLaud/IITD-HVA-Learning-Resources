# Server Setup Documentation

## Table of Contents
1. [Server Setup](#server-setup)
   - [Hostname Configuration](#hostname-configuration)
   - [Ubuntu Installation](#ubuntu-installation)
2. [Proxy Setup](#proxy-setup)
3. [Installing Packages](#installing-packages)
4. [Setting Remote Access through SSH](#setting-remote-access-through-ssh)
5. [Server Details](#server-details)
6. [KVM Pre-installation Checks](#kvm-pre-installation-checks)

---

## Server Setup

### Hostname Configuration
```bash
$ hostname
gem3-PowerEdge-R710
```

### Ubuntu 24.04 Installation
1. Restart the server.
2. Press `F11` to access the configuration menu and ensure UEFI boot is enabled.
3. Boot using a USB drive.
4. Erase the previous OS version while following installation prompts.

---

## Proxy Setup
1. Open a browser and navigate to **Settings**.
2. Search for "Proxy."
3. Set the Automatic Proxy Configuration URL to:
   ```
   http://www.cc.iitd.ac.in/cgi-bin/proxy.phd
   ```

![Proxy Setup Screenshot](https://github.com/user-attachments/assets/34d92e03-dfe8-4146-b909-aeb377a4d6ae)

---

## Installing Packages
1.	Package installation on Ubuntu attempts a connection to Ubuntu’s archives which is blocked in IITD.
2.	We need to set up an alternative path in `/etc/apt/sources.list` but that has changed in **Ubuntu24.04**. We have `/etc/apt/sources.list.d/ubuntu.sources` now.
3.	Change URIs in `/etc/apt/sources.list.d/ubuntu.sources` to http://repo.iitd.ernet.in/ubuntu/ but THIS IS NOT REQUIRED anymore.
4.	The corresponding change in /etc/apt/sources.list could have been: 
```

deb http://repo.iitd.ernet.in/ubuntu noble main restricted universe multiverse
deb http://repo.iitd.ernet.in/ubuntu noble-updates main restricted universe multiverse
deb http://repo.iitd.ernet.in/ubuntu noble-security main restricted universe multiverse
deb http://repo.iitd.ernet.in/ubuntu noble-backports main restricted universe multiverse

```
---

## Setting Remote Access through SSH
Ssh to my lab machine worked but ssh from lab machine to server did not work.
We needed to install `openssh-server`

1. First check if sshd is running:
```bash
   sudo netstat –anp | grep sshd  -- returned nothing
```

2. To update packages:
```bash
  sudo apt-get update
```

3. To install net-tools such as `netstat`:
```bash
  sudo apt install net-tools
```

4. To install ssh server:
```bash
  sudo apt install openssh-server
```
5. Allow SSH traffic:
```bash
  sudo ufw allow 22
```
Now ssh to the server started working but it is very slow.

---

## Server Details
### System Information
- **OS**: Ubuntu 20.04 (Noble)
- **Architecture**: x86_64 (64-bit)
- **CPU**: Intel Xeon 5600 series
```bash
uname: Linux
lspci: Host bridge: Intel Corporation Xeon 5600
lsb_release –a: Ubuntu 20.04 Noble
```
![System Architecture](https://github.com/user-attachments/assets/197e2a93-b6cb-47b3-86d3-9998088a2a30)

---

## KVM Pre-installation Checks
1. Verify hardware virtualization support:
   ```bash
   $ egrep -c '(vmx|svm)' /proc/cpuinfo
   0
   ```
   An output 0 means that the CPU does not support hardware virtualization
   
3. Check kernel:
   ```bash
   $ uname -a
   Linux gem3-PowerEdge-R710 6.8.0-22-generic #22-Ubuntu SMP PREEMPT_DYNAMIC Thu Apr 4 22:30:32 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux

   $ uname -r
   6.8.0-22-generic
   ```
4. Check if booted into Xen kernel:
   ```bash
   sudo dmesg | grep Xen
   sudo dmesg | grep xen
   ```
   Both return blank lines.
   
5. Additional Detailed Checks:

   - **Check for hypervisor properties:**
     ```bash
     $ cat /sys/hypervisor/properties/capabilities
     # Output: No such file or directory
     ```
     
   - **Verify KVM availability using kvm-ok:**
     ```bash
     $ kvm-ok
     Command 'kvm-ok' not found, but can be installed with:
     sudo apt install cpu-checker
     ```
     _After installing cpu-checker:_
     ```bash
     $ kvm-ok
     INFO: Your CPU does not support KVM extensions
     INFO: For more detailed results, you should run this as root
     HINT:   sudo /usr/sbin/kvm-ok
     ```
     _Note: According to the document, the message is misleading and only indicates that KVM is not available currently—it does not mean that it is not supported._
   
   - **Check if the CPU is 64-bit:**
     ```bash
     $ egrep -c ' lm ' /proc/cpuinfo
     16
     ```
     A non-zero value indicates that the CPU is 64-bit.
     
   - **Verify the running kernel is 64-bit:**
     ```bash
     $ uname -m
     x86_64
     ```
     _x86_64 is synonymous with amd64._
     
   - **Install necessary packages:**
     Attempting to install with:
     ```bash
     $ sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bride-utils
     ```
     _Output:_
     ```
	 Reading package lists... Done
	 Building dependency tree... Done
	 Reading state information... Done
     Note, selecting 'qemu-system-x86' instead of 'qemu-kvm'
     E: Unable to locate package bride-utils
     ```
     Retrying with:
     ```bash
     $ sudo apt-get install qemu-system-x86 libvirt-daemon-system libvirt-clients bride-utils
     ```
     _Output remains:_
     ```
	 Reading package lists... Done
	 Building dependency tree... Done
	 Reading state information... Done
     E: Unable to locate package bride-utils
     ```
     _After searching at packages.ubuntu.com, the package was found and installed independently. Then:_
     ```bash
     $ sudo apt-get update
     $ sudo apt install bridge-utils
     ```
   _Output:_
    ```bash
    Reading package lists... 
    Done Building dependency tree... 
    Done Reading state information... 
    Done Suggested packages: ifupdown 
    The following NEW packages will be installed: bridge-utils 0 upgraded, 
    1 newly installed, 0 to remove and 837 not upgraded. 
    Need to get 33.9 kB of archives. 
    After this operation, 118 kB of additional disk space will be used. 
    Get:1 http://repo.iitd.ernet.in/ubuntu noble/main amd64 bridge-utils amd64 1.7.1-1ubuntu2 [33.9 kB] 
    Fetched 33.9 kB in 0s (1,059 kB/s)
    Selecting previously unselected package bridge-utils. 
    (Reading database ... 150595 files and directories currently installed.) Preparing to unpack .../bridge-utils_1.7.1-1ubuntu2_amd64.deb ... 
    Unpacking bridge-utils (1.7.1-1ubuntu2) ... 
    Setting up bridge-utils (1.7.1-1ubuntu2) ... 
    Processing triggers for man-db (2.12.0-4build1) ...
    ```
   _Output confirms a successful installation of `bridge-utils`._
       
   - **Add user to the relevant groups (libvirt and kvm):**
     ```bash
     $ sudo adduser `id -un` libvirt
     ```
     _Output:_
     ```
     fatal: The group `libvirt' does not exist.
     ```
     Then:
     ```bash
     $ sudo adduser `id -un` kvm
     ```
     _Output:_
     ```
     info: Adding user `gem3' to group `kvm' ...
     ```
     And:
     ```bash
     $ sudo adduser `id -un` libvirtd
     ```
     _Output:_
     ```
     fatal: The group `libvirtd' does not exist.
     ```
     _Alternatively, add the user with:_
     ```bash
     sudo usermod –aG kvm $USER
     sudo usermod –aG libvirt $USER
     ```
     _To check available groups:_
     ```bash
     $ getent group
     ```
     _(No group similar to libvirt or libvirtd was initially found.)_
     To check all available groups reference: https://webhostinggeeks.com/howto/how-to-list-user-groups-on-ubuntu/
     
   - **Confirm KVM installation using virsh:**
     ```bash
     $ virsh list --all
     ```
     _Output:_
     ```
     Command 'virsh' not found, but can be installed with:
     sudo apt install libvirt-clients
     ```
     ![Installing libvirt](https://github.com/user-attachments/assets/19ffe735-1772-4187-b047-87920bde86dd)

     Installing the package, running again:
     ```bash
     $ virsh list --all
     ```
     _Output:_
     ```
     error: failed to connect to the hypervisor
     error: binary '/usr/sbin/libvirtd' does not exist in $PATH: No such file or directory
     ```
     _A YouTube video (https://www.youtube.com/watch?v=qCUmf5gyOYY) was consulted.
     Following its steps, virt-manager was installed:
     ![Installing virt-manager](https://github.com/user-attachments/assets/d55beecc-4500-4f21-a2d8-73869c6f9a6d)

     Checked groups _libvirt_ (using command `getent group`) was created and it appeared as if _gem3_ was already a member of the group.
     Still tried again and confirmed that gem3 is a member.

     After ensuring the user is a member of the libvirt group (confirmed via `getent group libvirt`), running:_
     ```bash
     $ virsh list --all
     ```
     _returned:_
     ```
     Id   Name   State
     --------------------
     ```
     
   - **Start the libvirtd daemon:**
     ```bash
     $ sudo systemctl enable libvirtd
     $ sudo systemctl start libvirtd
     ```
     _Then check its status:_
     ```bash
     $ sudo systemctl status libvirtd
     ```
     ![libvirtd command](https://github.com/user-attachments/assets/6df05e1e-0f90-4361-9862-cf21e736ac0b)

---
