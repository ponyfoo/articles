## What Are JSON Web Tokens?

JSON Web Token (JWT) is an open standard ([RFC 7519](https://tools.ietf.org/html/rfc7519)) that defines a compact and self-contained way to securely transmit information between parties as a JSON Object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with HMAC algorithm) or a public/private key pair using RSA.

## JWT Anatomy

JWTs basically consist of three parts separated by a `.` . This is the header, payload and signature. Check out this excellent [article](https://auth0.com/learn/json-web-tokens/) for a comprehensive explanation of the JWT Structure.
