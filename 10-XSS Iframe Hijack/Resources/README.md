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
