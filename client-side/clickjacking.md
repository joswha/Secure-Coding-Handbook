# Clickjacking

## 1. Introduction.

   **Clickjacking** is an **interface-based attack** in which a user is tricked into **clicking on overlayed actionable content** on a hidden website by clicking on some **content in a decoy website**.

Consider the following example: A web user **accesses a decoy website** and **clicks on a button to accept the cookie policy**. Unknowingly, this **decoy** website is overlayed with a button, that would, for example, remove their 2-factor authenticator, from the **vulnerable** website. This attack differs from a CSRF attack in that the user is required to **perform** an action **such as a button click** whereas a **CSRF attack depends upon forging an entire request** **without the user's knowledge or input whatsoever**.

{% hint style="success" %}
Read more about **Clickjacking**, from an [attacker's perspective](https://portswigger.net/web-security/clickjacking).
{% endhint %}

## 2. Defenensive techniques:

There are **no** effective ways of defending against clickjacking on the **client side**. With this in mind, we have to **properly setup the server side** so that our website will NEVER be a "**bait**" for such attacks.

### 2.1. Implementing X-Frame-Options or CSP\(frame-ancestors\):

The `X-Frame-Options` HTTP response header can be used to indicate whether or not a browser should be allowed to render a page in a `<frame>` or `<iframe>`. Sites can use this to avoid Clickjacking attacks, by ensuring that their content is not embedded into other sites. **Set** the **X-Frame-Options header for all responses containing HTML conten**t. Here are the possible **option** values:

* **DENY:**  prevents any domain from framing the content. The "DENY" setting is recommended unless a specific need has been identified for framing.
* **SAMEORIGIN:** which only allows the current site to frame the content.
* **ALLOW-FROM URI:** which permits the specified '**URI**' to frame this page.

### 2.2. Using a Content Security Policy\(CSP\):

 Content Security Policy \(CSP\) is a detection and prevention mechanism that provides mitigation against [**attacks such as XSS**](https://vladtoie.gitbook.io/secure-coding/client-side/xss#2-1-2-implementing-a-content-security-policy-csp) and **clickjacking**. CSP is usually implemented in the web server as a header of the form:**`Content-Security-Policy: policy`** where policy is a string of policy directives separated by semicolons. The CSP provides the client browser with information **about permitted sources of web resources** that the **browser can apply to the detection** and **interception** of malicious behaviors.

 The recommended **clickjacking** protection is to incorporate the **`frame-ancestors`** directive in the application's Content Security Policy. The **`frame-ancestors 'none'`** directive is similar in behavior to the X-Frame-Options **`deny`** directive. The **`frame-ancestors 'self'`** directive is broadly equivalent to the X-Frame-Options **`sameorigin`** directive, in the sense that they are pointing to the `self` domain. The following CSP whitelists frames to the same domain only:**Content-Security-Policy: frame-ancestors 'self';**

 Alternatively, framing can be restricted to named sites/ domains:**`Content-Security-Policy: frame-ancestors normal-website.com;`**

## 3. Conclusions:

Both `X-Frame-Options` and`Content-Security-Policy` response headers define whether or not a browser should be allowed to embed or render a page in an `<iframe>` element. As this topic is more related to server-side configuration rather than vulnerable code itself, here is a link to a [more "interactive" resource](https://application.security/free-application-security-training/owasp-top-10-clickjacking) where you can find how Clickjacking usually behaves. 

{% hint style="info" %}
You can find more details about this topic here:

* [Clickjacking Defense \[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html#defending-with-content-security-policy-csp-frame-ancestors-directive)
* [Clickjacking  \| Kontra exercise](https://application.security/free-application-security-training/owasp-top-10-clickjacking)
* [What is clickjacking?](https://portswigger.net/web-security/clickjacking)
{% endhint %}



