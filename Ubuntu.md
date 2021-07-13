## Enable Blue Tooth
```
$ sudo nano /etc/init.d/unblock-bluetooth

> #!/bin/bash
> rfkill unblock bluetooth
> systemctl restart bluetooth

$ sudo ln -s /etc/init.d/unblock-bluetooth /etc/rc2.d/S01unblock-bluetooth
```
