# Secure Code Review of OWASP's NodeGoat

## Summary
Performed a static review of the NodeGoat app for security vulnerabilities. Focused on authentication, input validation, access control, and common OWASP Top 10 issues. Used `semgrep` to speed up the static review.

## Manual Code Review
| Area | File | Potential Vulnerability | 
|---       |---       |---     |
| User Signup | routes/session.js | No input validation, weak password policy |
| User Signup | data/user-dao.js | Insecure password encryption, deprecated MongoDB Node.js functions |
| User Login | routes/session.js | No input validation/sanitization, poor error handling of invalid username and password, broken session management |
| Contributions | routes/contributions.js | Command injection |
| Allocations | routes/allocations.js | NoSQL injection, IDOR |
| Memos | routes/memos.js | No input validation - NoSQL injection, Stored XSS |
| User Profile | routes/profile.js | Insufficient encoding - Stored XSS, Regular Expression Denial of Service (ReDoS) - catastrophic backtracking, Insufficient input validation - DoS by HPP |
| Research | routes/research.js | No input validation - SSRF |

## Findings
### 1. Command Injection in Contributions page
- **Location**: `routes/contributions.js`, lines 32 - 34
- **Issue**: Use of `eval` function to parse inputs
- **Risk**: High — command injection and DoS possible
- **Fix**: Use `parseInt` function + input validation

### 2. Weak password policy in User Signup page
- **Location**: `routes/session.js`, line 144
- **Issue**: Use of `/^.{1,20}$/` regex for password policy
- **Risk**: Medium — brute force password attack
- **Fix**: Use `/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}$/` regex which allows for at least 8 characters with numbers and both lowercase and uppercase letters.

### 3. NoSQL injection in Allocations page
- **Location**: `routes/allocations.js`, lines 16-21
- **Issue**: Lack of input validation for `userId` and `threshold`
- **Risk**: Medium — access to server possible
- **Fix**: Use strict input validation `const userId = parseInt(req.params.userId, 10);
const threshold = parseFloat(req.query.threshold);`

### 4. Catastrophic Backtracking in User Profile page
- **Location**: `routes/profile.js`, line 59
- **Issue**: Use of `/([0-9]+)+\#/` regex for `bankRouting` input validation
- **Risk**: High — Regex Expression DoS
- **Fix**: Use `/([0-9]+)\#/` regex which removes greedy quantifiers

### 5. SSRF in Research page
- **Location**: `routes/research.js`, line 15
- **Issue**: The `url` parameter is built directly from user input (`req.query.url` + `req.query.symbol`)
- **Risk**: Medium — network scanning, information disclosure, privilege escalation
- **Fix**: Use a whitelist of allowed domains and validate the user input against that list
