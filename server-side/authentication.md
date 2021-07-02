# Authentication

## 1. Introduction:

**Authentication** plays a fundamental role in most applications nowadays, therefore it is also one of the juiciest picks for attackers. In the context of web applications, **Authentication** is traditionally performed through submitting a username or an email, along with a password\(which should, by common sense, be secret to each user\).

We are also going to delve into **Session Management** since it is a process that is tightly linked to modern authentication. Typically, sessions are maintained on the server-side, thus playing their role in recognizing how to handle the incoming requests. They are usually distinguished by **session identifiers**, which should be unique per user and session. It is important to mention here that good "**session ids**" are represented as large random numbers so that hackers can have as few changes in taking advantage of them. 

## 2. General guidelines:

### 2.1 Error messages:

Enforcing generic and identical error messages plays an important role in defending against any information leaks on your authentication system. A developer should also pay attention to using the same HTTP response codes.

Take a look at the following example:

{% tabs %}
{% tab title="NodeJS" %}
```javascript
...
// Validating the existence of a user with the specified email.
const existingUser = await User.findOne({ email });
if (!existingUser) {
    return res
        .status(401)
        .json({ errorMessage: "Invalid email." });
}

// Validating the password attributed to that User object with the passwordHash
// from the database.
const passwordCorrect = await bcrypt.compare(password, existingUser.passwordHash);
if (!passwordCorrect) {
    return res
        .status(401)
        .json({ errorMessage: "Password is invalid for the given email." });
}
...
```
{% endtab %}
{% endtabs %}

The code above is clearly a **no-go**. The attacker can observe the variation of error messages and start a brute-force attack that could possibly leak some existing emails from the target database.

In order to properly sanitize our errors, we should provide as little information as possible while still letting the user know that a mistake occurred, so that they can take further action. 

A very good alternative could be:

{% tabs %}
{% tab title="NodeJS" %}
```javascript
...
// Validating the existence of a user with the specified email.
const existingUser = await User.findOne({ email });
if (!existingUser) {
    return res
        .status(401)
        .json({ errorMessage: "Invalid email or password." });
}

// Validating the password attributed to that User object with the passwordHash
// from the database.
const passwordCorrect = await bcrypt.compare(password, existingUser.passwordHash);
if (!passwordCorrect) {
    return res
        .status(401)
        .json({ errorMessage: "Invalid email or password." });
}
...
```
{% endtab %}
{% endtabs %}

### 2.2. Username and emails:

Generally, make sure your usernames/user IDs are case-insensitive. User 'bobi' and user 'Bobi' should refer to the same user account. This can be enforced through "lower-casing" the user input before storing it into the database. You could even go as far as assigning a randomly generated username, thus providing some sort of secure data, instead of the user-defined one.

{% hint style="warning" %}
It is important to **NEVER** allow login to sensitive accounts\(i.e company's internal accounts\).
{% endhint %}

Should it really be the case of needing to display your user's identification publicly, such as a username, what you can do instead is create a **Display Name** field to their profile/ account, such that you **NEVER** leak the username itself.

#### **2.2.1. Validating emails:**

If you intend to send an email from your site, without first-hand knowing whether or not that email is a valid one, you need to validate that address to check whether it corresponds to an email account that is actually working. Sending emails to unverified addresses, such as the ones that you can temporarily generate and use online, will[ quickly get you blacklisted by your email service provider](https://mailtrap.io/blog/python-validate-email/) since they’re suspicious of enabling spamming.

By first validating the format of the email, and then looking up the MX record for the given domain, we can easily set up a very efficient email validator. This is where the **email verification link** plays its role. To be 100% sure that an email certainly is valid, a developer should send one of those **"Verify your account"** emails, which should then be further verified using a token that is unique for each newly created account.

```python
import dns.resolver
import re

def checkFormat(email):
  #email regex 
  regex = '\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b'
  if re.search(regex, email):
    return True
  else:
    return False

def checkEmailValid(email):
  if checkFormat(email):
    domain = email.split("@")[1] # 'bobi.io'
    for _ in dns.resolver.query(domain, 'MX'):
      return True
  return False

email = "vlad@bobi.io"
checkEmailValid(email) # True
```

### 2.3. Passwords:

It goes without saying that passwords should be **STRONG.** I assume you are already familiar with most PASSWORD good-practices rules, however, here's a list of the most important to follow:

* Setting a **MINIMUM** **LENGTH** of at least 8-10 characters is essential. Anything less is considered unreliable. On the side, you can also set a **MAXIMUM LENGTH**  of about 60 characters, since most probably the [hashing algorithm that you will use to securely store that password](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#maximum-password-lengths) has a limit at around 64 characters. 
* Allow usage of any characters\( including Unicode and whitespace\).
* You can even try using the [HaveIBeenPwned API](https://github.com/fvdm/nodejs-haveibeenpwned), to check whether the inputted password has been breached or not.

```javascript
const hibp = require ('haveibeenpwned')();

hibp.pwnedpasswords.byPassword ('securepassword', (err, count) => {
  if (!count) {
    console.log ('Great! Password is not found.');
  } else {
    console.log ('Oops! Password was found ' + count + ' times!');
  }
});
```

#### 2.3.1. Forgot your password?

As previously mentioned, error messages provide an important layer of defense against ill-intended attackers, which is often overlooked.  With this in mind, it's important to consider the following response messages.

Instead of: **"We just sent you a password reset link."**

You can go with: "**If that email address is in our database, we will send you an email to reset your password."**

More on such error messages can [be read here](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#password-recovery) and [**here**](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).

### 2.4. CAPTCHA:

One of your first preventions against brute-forcing is CAPTCHA if used in an effective way. Since hackers have already found workarounds for some CAPTCHAs, this technique should be used as delaying the timespan of an attack, rather than preventing it altogether.

### 2.5. Logging and monitoring:

Enable logging and monitoring of authentication functions to detect attacks/failures on a real-time basis.

* **Ensure** that all **failures** are logged and reviewed.
* **Ensure** that all **password failures** are logged and reviewed.
* **Ensure** that all **account lockouts** are logged and reviewed.

## 3. Conclusions:

As of right now, the most important aspects of **properly handing authentication are**: making sure that **error messages illustrate as little and concise information as possible**, **properly validating** and even **checking the existance** of an email address, and **enforcing a "secure password" policy** such that there is little room for a bruteforce attack.

{% hint style="info" %}
You can find more details about this topic here:

* [Authentication Cheat Sheet \[OWASP\].](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
* [Authorization Cheat Sheet \[OWASP\].](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
* [Choosing and Using Security Questions \[OWASP\].](https://cheatsheetseries.owasp.org/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.html)
* [Secure Password Storage \[OWASP\].](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
{% endhint %}

