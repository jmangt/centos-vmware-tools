## Overview
This procedure enables support of the Vagrant synced folders using VMware's native shared folder support

## Requirements
* Latest version of VMware Fusion Pro (8.5) or Workstation Pro (12.5). Other versions are untested but should work
* Latest version of Vagrant (1.8.6) and Vagrant VMware plugin (4.0.13)
* Additional steps may be required if the kernel in the Vagrant guest is updated

## Host preparation
* Copy the VMware Tools installer from the VMware application folder to the guest. The default locations for the installer are:
  * `/Applications/VMware\ Fusion.app/Contents/Library/isoimages/linux.iso` (OS X)
  * `C:\Program Files (x86)\VMware\VMware Workstation\linux.iso` (Windows)
* Use `vagrant rsync` (OS X) or a SMB synced folder (Windows) to copy the iso into the guest's /vagrant directory

## Commands to run within Vagrant guest
#### Install the EPEL repository (CentOS 6 only)
```
sudo yum -y install epel-release
```

#### Install the open source tools package
This provides all VMware functionality except for the HGFS driver required for synced folders:
```
sudo yum -y install open-vm-tools
```

#### Install build dependencies
```
sudo yum -y install perl gcc fuse make kernel-devel net-tools policycoreutils-python
```

#### Mount the installer iso and extract it to /tmp
```
mkdir -p /tmp/vmware /tmp/vmware-archive
sudo mount -o loop /vagrant/linux.iso /tmp/vmware
TOOLS_PATH="`ls /tmp/vmware/VMwareTools-*.tar.gz`"
tar xzf ${TOOLS_PATH} -C /tmp/vmware-archive
```

#### Install VMware Tools
Both --force-install and --default are required to accept all default options, as we have already installed open-vm-tools. See https://kb.vmware.com/kb/2126368 for more details:
```
sudo /tmp/vmware-archive/vmware-tools-distrib/vmware-install.pl --force-install --default
```

#### Cleanup
```
sudo umount /tmp/vmware
rm -rf /tmp/vmware /tmp/vmware-archive /vagrant/*.iso
# linux.iso can also be removed from the host
```

## Create a new shared folder using VMware shared folders
It currently isn't possible to override the default rsync synced_folder for /vagrant (this a known issue). As a workaround, use a different name for the mount point in the guest, by adding a line like this to the Vagrantfile before the final `end`:
```
config.vm.synced_folder ".", "/vagrant2"
```

## Restart the VM and test
```
vagrant reload
```

## References:
* http://partnerweb.vmware.com/GOSIG/CentOS_7.html
* https://github.com/chef/bento/blob/master/scripts/common/vmware.sh
