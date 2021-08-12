# Open Redirects

## 1. Introduction:

Insecure redirects and forwards are possible when a web application redirects the request to an URL given by accepts arbitrary input. By modifying the arbitrary URL input to a malicious site, an attacker may successfully launch a phishing scam and steal user credentials, and much more. For more information regarding this, from an attacker's perspective, [you can read this](https://cwe.mitre.org/data/definitions/601.html).

## 2. Typical vulnerable code:

The typical vulnerable code takes in some `url` parameter to which it redirects the user.

{% tabs %}
{% tab title="PHP" %}
```php
<?php
...
$redirect_url = $_GET['url'];
header("Location: " . $redirect_url);
...
?>
```
{% endtab %}

{% tab title="Java" %}
```java
...
response.sendRedirect(request.getParameter("url"));
...
```
{% endtab %}
{% endtabs %}

## 3. Mitigations:

The rule of thumb that you have probably noticed by now, should you have read through all other topics, is the fact that every developer has to **VALIDATE their input!** Input validation and filtering are the techniques of protecting against any kind of **user-available-input** exploitation. 

In this scenario, however, input validation is not even needed. Since a developer typically knows where they have to redirect the user to, there is no longer a need for the `url` parameter that can be exploited. We can just hardcore it :\)

On the `Java` tab of the following code brick you will find a way to not hardcore the `url` itself, but to construct it instead.

{% tabs %}
{% tab title="PHP" %}
```php
<?php

/* Redirect the browser. */
header("Location: http://www.bobi.io");

/* Exit to prevent the rest of the code from executing. */
exit;

?>
```
{% endtab %}

{% tab title="Java" %}
```java

// build the redirectURL based on the hostname lookup
StringBuilder redirectUrl = new StringBuilder()
    .append(request.getScheme())
    .append("://")
    .append(InetAddress.getLoopbackAddress().getHostName())
    .append(request.getContextPath()
);

//
response.sendRedirect(redirectUrl);
```
{% endtab %}
{% endtabs %}

## 4. Takeaways:

Safe use of redirects and forwards can be done in a number of ways:

* Simply avoid using redirects and forwards.
* If used, **do not allow the URL** as user input for the destination.
* If user input can’t be avoided, ensure that the supplied **value** is valid, appropriate for the application, and is **authorized** for the user.
* Sanitize input by creating a list of trusted URLs \(lists of hosts or a regex\).
  * This should be based on an **allow-list approach,** **rather than a block list.**
* **Force all redirects to first go through a page** notifying users that **they are going off of your site**, with the destination clearly displayed, and have **them click a link to confirm.**

{% hint style="info" %}
You can find more details about this topic here:

* [Open redirection \(reflected\)](https://portswigger.net/kb/issues/00500100_open-redirection-reflected)
* [Insecure URL Redirect](https://application.security/free-application-security-training/owasp-top-10-insecure-url-redirect)
* [Unvalidated Redirects and Forwards](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html)
{% endhint %}

