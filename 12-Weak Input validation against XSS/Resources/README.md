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
