# Exploit: (mistake) unsafe redirection
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