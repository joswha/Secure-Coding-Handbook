# Host Header Injection

## 1. Introduction:

 **HTTP Host header attacks** exploit vulnerable websites that handle the value of the Host header in an unsafe way. If the server implicitly trusts the Host header, and fails to validate or escape it properly, an attacker may be able to use this input to inject harmful payloads that manipulate server-side behavior. Attacks that involve injecting a payload directly into the Host header are often known as "**Host header injection**" attacks.

 **Off-the-shelf web applications** typically don't know what domain they are deployed on **unless it is manually specified in a configuration file during setup**. 

 As the **Host header is in fact user controllable**, this practice can lead to a number of issues. If the input is not properly escaped or validated, the Host header is a potential vector for exploiting a range of other vulnerabilities, most notably:

*  Web cache poisoning.
*  Business [logic flaws](https://portswigger.net/web-security/logic-flaws) in specific functionality.
*  Routing-based SSRF.
*  Classic server-side vulnerabilities, such as SQL injection.

Read more about this topic, from [the attacker's perspective](https://portswigger.net/web-security/host-header).

## 2. Typical vulnerable code:

Password resetting represents one of the most common targets of **Header Injection;** here's an example of how vulnerable code looks like:

```java
public void resetPasswordLink(HttpServletRequest request) {

    // retrieves the host from the request header
    String host = request.getHeader("Host");
    
    String email = request.getParameter("email");
    HttpSession session = request.getSession();

    if (session != null) {
        String token = generateResetToken(email);
        
        // Password reset link is constructed with the retrieved host
        // for the token that has just been generated.
        StringBuilder resetLinkBuilder = new StringBuilder()
                .append(host)
                .append("?reset")
                .append(token);
        
        // Send the email
        sendEmail(email, resetLinkBuilder.toString());
    }
}
```

Note that there is no filtering whatsoever on the **`Host`** header. An _attacker_ **can take advantage** of this and intercept the **`POST`** request to arbitrarily modify the **`Host`** from the request header. Thus, the email that the _victim_ will receive would look safe, however upon accessing the link that was generated\(with the attacker's **`Host`**\), the _attacker_ will receive the **`token`** of the password reset, hence being able to completely reset the _victim's_ password.

## 3. Mitigations:

 To prevent HTTP Host header attacks, the simplest approach is to avoid using the **`Host`** **header** altogether in server-side code. Double-check **whether each URL really needs to be absolute**. You will often find that you can just use a **relative URL instead**. This simple change can help you **prevent** [**web cache poisoning**](https://portswigger.net/web-security/web-cache-poisoning) vulnerabilities in particular.

### 3.1. **Whitelist permitted domains**:

Probably the easiest technique is creating a whitelist of possible domains from where you would typically send the request from. This, of course, **should only be used in the case** that it is in your application's intended behavior to send the specific request from various domains.

```java
public void resetPasswordLink(HttpServletRequest request) {

    // retrieves the host from the request header
    String host = request.getHeader("Host");
    
    String email = request.getParameter("email");
    HttpSession session = request.getSession();
    
    // create the whitelist for the allowed domains
    List <String> allowedDomains = Arrays.asList(
        "bobisecure.com",
        "bank.bobisecure.com",
        "s3.bobisecure.com"
    );
    
    // check whether the given `host` exists in the whitelist
    if (session != null && allowedDomains.contains(host)) {
        String token = generateResetToken(email);
        
        // Password reset link is constructed with the retrieved host
        // for the token that has just been generated.
        StringBuilder resetLinkBuilder = new StringBuilder()
                .append(host)
                .append("?reset")
                .append(token);
        
        // Send the email
        sendEmail(email, resetLinkBuilder.toString());
    }
}
```

### **3.2. Protect absolute URLs:**

 When you have to use absolute **URLs**, you should require the current domain to be manually specified in a configuration file and refer to this value instead of the Host header. Contrary to the previous point, your application's endpoint would only deal with a singular possible **URL.** This approach eliminates the threat of password reset poisoning, for example.

{% hint style="danger" %}
 It is also **important** to check that you do not support **additional headers** that may be used to construct these attacks, in particular **`X-Forwarded-Host`**. Remember that these may be **supported by default**.
{% endhint %}

## 4. Takeaways:

Taking the simple yet often overlooked points that we have discussed here should protect your application against any type of Host Header Injections.

 Remember that all in all, you should never trust the `Host` header unless it is really necessary!

{% hint style="info" %}
You can find more details about this topic here:

* [HTTP Host Header Attacks](https://portswigger.net/web-security/host-header)
* [Host Header Injection Training](https://application.security/free-application-security-training/owasp-top-10-host-header-injection)
* [What is a Host Header Attack?](https://dzone.com/articles/what-is-a-host-header-attack)
{% endhint %}

