---
layout: post
title: Best Practices in Web Application Security
author: Pedro Ruiz Pareja
date: 2024-09-20 13:00:37 +0200
categories: security
tags: web security, best practices, code examples
---

Web application security is one of those things that, like flossing or backing up your data, everyone agrees is important but not everyone actually does. And just like with flossing or backups, the consequences of not paying attention can be pretty severe. So let's talk about some best practices in web application security. I'll try to keep it practical and sprinkle in some code examples.

## 1. Use HTTPS Everywhere

This seems obvious, but you'd be surprised how many sites still don't use HTTPS. HTTPS encrypts the data between the user's browser and your server, making it much harder for attackers to intercept sensitive information.

To set up HTTPS, you'll need an SSL/TLS certificate. You can get one for free from [Let's Encrypt](https://letsencrypt.org/). Here's how you might configure your Nginx server:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 2. Input Validation and Sanitization

Input validation is about ensuring the data your application receives is what you expect. Sanitization is about cleaning that data to remove any potentially harmful content. Never trust user input.

For example, if you're using Node.js and Express, you can use middleware like `express-validator` to handle validation:

```javascript
const { body, validationResult } = require('express-validator');

app.post('/submit-form', [
    body('email').isEmail(),
    body('password').isLength({ min: 5 })
], (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
    }
    // Proceed with processing the form
});
```

## 3. Use Prepared Statements

SQL injection is one of the oldest tricks in the book, but it still works surprisingly often. The best way to prevent it is by using prepared statements, which ensure that SQL code and data are never mixed.

Here's an example using Python's `sqlite3` library:

```python
import sqlite3

conn = sqlite3.connect('example.db')
cursor = conn.cursor()

# Unsafe
cursor.execute(f"SELECT * FROM users WHERE username = '{username}'")

# Safe
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
```

## 4. Implement Proper Authentication

Passwords should be hashed and salted before being stored in your database. Never store passwords as plain text.

Here's how you might do it in Python using the `bcrypt` library:

```python
import bcrypt

# Hash a password
password = b"supersecretpassword"
hashed = bcrypt.hashpw(password, bcrypt.gensalt())

# Check a password
if bcrypt.checkpw(password, hashed):
    print("Password match")
else:
    print("Password does not match")
```

## 5. Use Multi-Factor Authentication (MFA)

Multi-factor authentication adds an extra layer of security by requiring not just a password but also something else, like a code sent to your phone.

You can implement MFA using libraries like `pyotp` for generating one-time passwords:

```python
import pyotp

# Generate a base32 secret
secret = pyotp.random_base32()

# Create a TOTP object
totp = pyotp.TOTP(secret)

# Get the current OTP
otp = totp.now()

print("Current OTP:", otp)
```

## 6. Secure Cookies

Cookies can be a weak point if not handled properly. Always use the `HttpOnly` and `Secure` flags to protect your cookies.

Here's how you might set a cookie in Express:

```javascript
app.get('/set-cookie', (req, res) => {
    res.cookie('session', '123456', {
        httpOnly: true,
        secure: true,
        sameSite: 'strict'
    });
    res.send('Cookie is set');
});
```

## 7. Regularly Update Dependencies

Outdated dependencies can be a significant security risk. Make sure to regularly update your dependencies to patch any vulnerabilities.

For Node.js, you can use `npm-check-updates` to see which dependencies need updating:

```sh
npm install -g npm-check-updates
ncu -u
npm install
```

## 8. Use Content Security Policy (CSP)

CSP helps prevent cross-site scripting (XSS) attacks by specifying which dynamic resources are allowed to load.

Here's an example of setting a CSP header in Express:

```javascript
app.use((req, res, next) => {
    res.setHeader("Content-Security-Policy", "default-src 'self'");
    return next();
});
```

## 9. Limit Rate of Requests

Rate limiting can help prevent abuse and brute-force attacks by limiting the number of requests a user can make in a given time period.

For example, using `express-rate-limit`:

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

## 10. Regular Security Audits

Regular security audits can help you find and fix vulnerabilities before they are exploited. Consider using automated tools like `OWASP ZAP` or `Burp Suite`.

You can also perform manual code reviews and penetration testing to identify potential issues.

## References

1. Let's Encrypt. (n.d.). Retrieved from https://letsencrypt.org/
2. Express Validator. (n.d.). Retrieved from https://express-validator.github.io/docs/
3. OWASP. (n.d.). Retrieved from https://owasp.org/
4. bcrypt. (n.d.). Retrieved from https://pypi.org/project/bcrypt/
5. pyotp. (n.d.). Retrieved from https://pypi.org/project/pyotp/

Security is a moving target. As new threats emerge, best practices evolve. But by following these principles and staying informed, you can keep your web applications as secure as possible. Happy coding!
