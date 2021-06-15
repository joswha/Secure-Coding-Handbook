# Cross-Site Request Forgery \[CSRF\]

## 1. Introduction:

This attack occurs when a malicious website, email, blog, instant message, or program causes a user's web browser to perform an unwanted action on a trusted site when the user is authenticated. Imagine the following scenario: an attacker crafts a malicious link into the target\(vulnerable\) website; this vulnerable website allows for a component that executes a `GET` request to a resource, for example: `http://bobi.io/load?page=http://funnyimage.com` . As we mentioned, the attacker crafted a malicious link, such as `http://bobi.io/load?page=http://malicious.io`which could thus expose the user's account information, such as session cookies and so on from the `bobi.io` vulnerable page; this works because browser requests include all cookies by default.

Since `GET` requests are the only type of HTTP requests that contain the entirety of the request's contents in the URL, they are uniquely vulnerable to this type of attacks. You can read more about this type of attack [here](https://portswigger.net/web-security/csrf).



[https://portswigger.net/web-security/csrf](https://portswigger.net/web-security/csrf)

