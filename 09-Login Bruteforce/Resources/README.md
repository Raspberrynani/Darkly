
## Exploit: Very Easy Bruteforce in login

### Enumeration
The login page seems easily bruteforcable, Looking at the admin page bruteforcing as refrence

### Solution
I ran a script that bruteforced the common username and password, it took less than 10 hrs on a 3090 and 7950X,(i went to sleep lol), and the username and password is admin, shadow
![image](https://hackmd.io/_uploads/Sk01yibXgx.png)
![image](https://hackmd.io/_uploads/SyzzkiZXxx.png)

> The flag is `B3A6E43DDF8B4BBB4125E5E7D23040433827759D4DE1C04EA63907479A80A6B2`

### Mitigation
Common password should be avoided, use number and symbols
