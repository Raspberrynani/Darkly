## Exploit: exposed htpasswd file
In the `/whatever` directory, there is a plaintext file called `htpasswd` file being exposed.

The name of this file corresponds to the [htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) service by apache for authentication. The contents of the file are like so:

![image](https://hackmd.io/_uploads/By9DkMemll.png)

The list of encryption methods the service uses are MD5, SHA-1x or SHA-2x. 

> htpasswd hashes passwords using either bcrypt, a version of MD5 modified for Apache, SHA-1, or the system's crypt() routine. SHA-2-based hashes (SHA-256 and SHA-512) are supported for crypt(). Files managed by htpasswd may contain a mixture of different encoding types of passwords; some user records may have bcrypt or MD5-hashed passwords while others in the same file may have passwords hashed with crypt().

Upon placing the ciphertext in a cipher detector, the MD5 cipher was its strongest suggestion:

![image](https://hackmd.io/_uploads/r1xWxfe7le.png)

The online decryptor and it does not work, which leads us to manual bruteforcing. There is a tool called [hashcat](https://www.kali.org/tools/hashcat/#hashcat-1) which can help us with this. Hashcat is a bruteforcing tool with extra functionality which includes temperature monitoring, masking, checkpointing and so on. It also supports the MD5 hash bruteforce.

A pure bruteforce was attempted up to 8 characters with alphanumeric characters only and we did not find a result. 

However, the attempt with the [rockyou](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) wordlist is successful, and we managed to recover the password `qwerty123@`

![image](https://hackmd.io/_uploads/BkLCqMe7gg.png)

Entering the password in the `/admin` page, we get our flag
`d19b4823e0d5600ceed56d5e896ef328d7a2b9e7ac7e80f4fcdb9b10bcb3e7ff`

![image](https://hackmd.io/_uploads/ry4LhfeXee.png)

> The flag is `d19b4823e0d5600ceed56d5e896ef328d7a2b9e7ac7e80f4fcdb9b10bcb3e7ff`

### Mitigations
- Same as .hidden/ directory exploit
- Do not use common passwords
- Always validate passwords against wordlists upon creation
- Admin page should not be served in the same host as the user facing application (enumeration fault)
