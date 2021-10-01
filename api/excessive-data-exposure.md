# Excessive Data Exposure

## 1. Introduction:

This vulnerability happens when the API relies on clients to perform the **data filtering**. Because APIs are thought of as **data sources**, sometimes developers try to implement them in a **generic way** without thinking about the **sensitivity of the exposed data**. It is essential to **only reveal precise details about your data**, even when **dealing with error messages**, such that the **risk of exposing any unwanted data** is minimized **on your behalf**.

## 2. Typical vulnerable code:

In this `forgot-password` scenario, the developer sends the **entire** `resetData` information to the **client**.

```javascript
/*

Imagine that resetData looks like:

resetData: {
    "resetToken": "59f3efe43ad4cdc8b8cfc0927a3d4147",
    "email": "bobi@bob.io",
    "lastLoggedIn": "25.03.1919",
    ...
}
*/

router.post('/forgot-password', (req, res) => {
    
    const email = req.body.email;
    const isValidEmail = Email.validate(email);

    if (isValidEmail) {
        const resetData = User.generatePasswordReset(email);
        Email.sendPasswordReset(email, resetData.resetToken);
        res.status(200).send(resetData);
    } else {
        res.status(400).send("The email address that you have provided is invalid.");
    }
});

```

Although the developer sends the `resetData.resetToken` to the **given emai**l, the entire `resetData` object is **also being sent to the client**, thus **leaking fields that should not be reachable** by the end-user.

## 3. Mitigation:

To mitigate this, a developer should **thoroughly consider their response messages and the content within**, regardless of the endpoint. For the vulnerable scenario above, here is how to mitigate it:

```javascript
router.post('/forgot-password', (req, res) => {
    
    const email = req.body.email;
    const isValidEmail = Email.validate(email);

    if (isValidEmail) {
        const resetData = User.generatePasswordReset(email);
        Email.sendPasswordReset(email, resetData.resetToken);
    }

    res.status(200).send("Password reset link has been successfully sent! Check your inbox.");
});
```

Our **objective** was to create a `resetToken` and send it to the given email. Do not try and extend a method's functionality! Everything should be **straight to the point.**

```javascript
 Email.sendPasswordReset(email, resetData.resetToken);
```

Note that we removed the **error handling** in the case that the email address is invalid. This is simply because the end-user should **never know** whether the email they have provided exists in our database or not, as this can be further exploited into **email enumeration**. More on authentication logic here:

{% page-ref page="../server-side/authentication.md" %}

## 4. Takeaways:

In short, make sure to follow that:

* **Never rely on the client-side** to filter sensitive data; this should be done at the API level.
* Cherry-pick the specific properties of the response you want to return.

{% hint style="info" %}
You can find more details about this topic here:

* [Excessive Data Exposure](https://raw.githubusercontent.com/OWASP/API-Security/master/2019/en/dist/owasp-api-security-top-10.pdf)
* [CWE-200: Exposure of Sensitive Information to an Unauthorized Actor](https://cwe.mitre.org/data/definitions/200.html)
{% endhint %}

