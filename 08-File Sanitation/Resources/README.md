
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
