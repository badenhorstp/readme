## Rename batch files in Bash
```
$ ls -1 Phoenix* | awk '{print("mv ./"$1 " ./"$1)}' | sed 's|.7z||4' | sh
```