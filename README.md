# Secure Code Review of OWASP's NodeGoat

## Manual Code Review
| Area | File | Potential Vulnerability | 
|---       |---       |---     |
| User Signup | routes/session.js | No input validation, weak password policy |
| User Signup | app/data/user-dao.js | Insecure password encryption, deprecated MongoDB Node.js functions |
