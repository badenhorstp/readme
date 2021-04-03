## 1. Install OpenJDK 8
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install git -y
sudo apt install openjdk-8-jdk
nano ~/.bashrc
+ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```
## 2. Install Android SDK Command Line Tools
```bash
wget https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip
unzip commandlinetools-linux-6858069_latest.zip
mkdir ~/android-sdk/cmdline-tools
mv cmdline-tools ~/android-sdk/cmdline-tools/3.0
```

## 3. Install Platform and Build Tools
```bash
sdkmanager "platform-tools" "platforms:android-29" "build-tools;29.0.3"
sdkmanager --uninstall emulator

nano ~/.bashrc
+export ANDROID_HOME=$HOME/android-sdk
+export PATH=$PATH:$ANDROID_HOME/cmdline-tools/3.0/bin:$ANDROID_HOME/platform-tools

```

## 4. Install Flutter
```bash
wget https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_2.0.3-stable.tar.xz
tar xf flutter_linux_2.0.3-stable.tar.xz
sudo mv flutter /opt
nano ~/.bashrc
^export PATH=$PATH:$ANDROID_HOME/cmdline-tools/3.0/bin:$ANDROID_HOME/platform-tools:/opt/flutter/bin
```

## 5. Install Chrome
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

## 6. Install VSCode
```bash
wget https://az764295.vo.msecnd.net/stable/2b9aebd5354a3629c3aba0a5f5df49f43d6689f8/code_1.54.3-1615806378_amd64.deb
sudo dpkg -i 2b9aebd5354a3629c3aba0a5f5df49f43d6689f8/code_1.54.3-1615806378_amd64.deb
```

## 7. Install Python and Pip
```bash
sudo apt install python3 python3-pip -y
```

## 8. Install Python VirtualEnv (Optional)
```bash
pip3 install virtualenv
pip3 install virtualenvwrapper
mkdir ~/.virtualenvs
# virtualenv ~/.virtualenvs
export WORKON_HOME=~/.virtualenvs
VIRTUALENVWRAPPER_PYTHON='/usr/bin/python3'
source $HOME/.local/bin/virtualenvwrapper.sh
```
