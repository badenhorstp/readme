## Enable Blue Tooth
```
$ sudo nano /etc/init.d/unblock-bluetooth

> #!/bin/bash
> rfkill unblock bluetooth
> systemctl restart bluetooth

$ sudo ln -s /etc/init.d/unblock-bluetooth /etc/rc2.d/S01unblock-bluetooth
```
## Enable FireFox for Touch Screen
```
$ echo export MOZ_USE_XINPUT2=1 | sudo tee /etc/profile.d/use-xinput2.sh
```
## Enable display fractional scaling
```
$ gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

## Install and sign kernel
* https://gloveboxes.github.io/Ubuntu-for-Azure-Developers/docs/signing-kernel-for-secure-boot.html
* https://wiki.ubuntu.com/UEFI/SecureBoot/Signing
* https://ubuntu.com/blog/how-to-sign-things-for-secure-boot
* https://linuxconfig.org/how-to-upgrade-kernel-to-latest-version-on-ubuntu-20-04-focal-fossa-linux

## Manage EFI boot items
* https://www.linuxbabe.com/command-line/how-to-use-linux-efibootmgr-examples
