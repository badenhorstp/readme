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
