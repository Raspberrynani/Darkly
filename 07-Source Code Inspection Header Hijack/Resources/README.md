
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
