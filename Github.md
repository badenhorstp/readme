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
Push to upstream remote repository
```bash
$ git push --set-upstream origin master
```


# Merge Github Repositories
## Empty remote repository
```bash
$ echo 'Readme file' > README.md
$ git init
$ git add .
$ git commit -m 'Initial commit'
$ git branch -M main    # Rename branch to remote branch name or
                        # set global config (git config --global init.
                        # defaultBranch=<name>)(git version >= 2.28)
$ git remote add origin git@github.com:myname/repositoryname.git
$ git push -u origin main
```

## Empty local repository
```bash
$ git init
$ git remote add origin git@github.com:myname/repositoryname.git
$ git pull --set-upstream orign main

```

## Local and remote have branches
```bash
$ git int
$ echo 'file0' > file0.txt
$ git add .
$ git commit -m 'Added file0.txt'
$ got remote add origin git@github.com:myname/repositoryname.git
$ git pull --rebase --set-upstream origin main
$ git push
```
##### TODO
* Configure developer PowerShell in Visual Studio Code
* Configure OpenSSH Agent on Windows
* Connecting to GitHub with SSH
