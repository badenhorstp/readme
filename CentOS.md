## Change Yum/Dnf repository
```bash
$ nano /etc/yum.repo.d/CentOS-AppStream.repo
+ baseurl=http://ftp.is.co.za/mirror/centos/$releasever/AppStream/$basearch/os/

$ nano /etc/yum.repo.d/CentOS-Base.repo
+ baseurl=http://ftp.is.co.za/mirror/centos/$releasever/BaseOS/$basearch/os/

$ nano /etc/yum.repo.d/CentOS-Extras.repo
+ baseurl=http://ftp.is.co.za/mirror/centos/$releasever/extras/$basearch/os/

```
## Install GNOME on CentOS 8
1. Install GNOME
```bash
$ dnf groupinstall GNOME -y
$ systemctl set-default graphical.target
```
2. Install X11 driver for VMWare client
```bash
$ dnf install xorg-x11-drv-vmware open-vm-tools-desktop -y

```

