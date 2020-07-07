# Documentation

## Connecting to GitHub with SSH
### Generate a new SSH key
```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
### Add SSH key to the ssh-agent
```
ssh-add [path home/profile directory]/.ssh/id_rsa
```

## Add remote Git repository to local repository
### Initialize local repository
```bash
$ git init
```
### Add remote repository
```bash
$ git remote add orgin [remote repository]
```
### Push to remote repository to local repository
```bash
$ git push -u origin master
```
### Add local content
```bash
$ git add .
```
### Commit changes
```bash
$ git commit -m [Message]
```
### Push to upstream remote repository
```bash
$ git push --set-upstream origin master
```

##### TODO
* Configure developer PowerShell in Visual Studio Code
* Configure OpenSSH Agent on Windows
* Connecting to GitHub with SSH
