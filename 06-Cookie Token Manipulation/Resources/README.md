
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

