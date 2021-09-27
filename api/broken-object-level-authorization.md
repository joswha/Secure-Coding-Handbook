# Broken Object Level Authorization

## 1. Introduction:

**Broken Object Level Authorization** happens **when an application does not correctly confirm that the user performing the request has the required privileges to access the user they are requesting for**. Since APIs typically expose endpoints that handle some kind of object identifiers\(ids, names, tags, etc\) they allow for a wide attack surface of improper access.

## 2. Typical vulnerable code:

Imagine that this endpoint is used when users want to retrieve their personal secrets. 

```javascript
const express = require('express');
const UserHandler = require('UserHandler');
const SecretsHandler = require('SecretsHandler');

const app = express();
const router = app.router;

router.get('/api/:userId/secrets', (req, res) => {
    const userId = req.params.userId;

    const user = UserHandler.getUserDetails(userId);
    const secrets = SecretsHandler.getSecretsByUser(user);

    return res.status(200).send({
        user,
        secrets,
    });
});

//...
```

 We observe that the API does not check whether the user doing the actual request is indeed the user that the secrets have been requested for. _\(line 11 and 12\)._ Due to this fault, an attacker with `userId = 1337`, for example, can retrieve the secrets of the account with `userId=1`, simply by altering the request query.

## 3. Mitigations:

### 3.1. Access control:

Implementing a strict access control policy is essential in this case. A developer should deal with session management instead of allowing an arbitrary parameter querying the API. Since sessions are typically more secure and \(should\) be really hard to tamper with, it is by far the best choice when securing object referencing endpoints.

```javascript
const express = require('express');
const UserHandler = require('UserHandler');
const SecretsHandler = require('SecretsHandler');

const app = express();
const router = app.router;

router.get('/api/secrets', (req, res) => {

    const userId = req.session.userId;
    if (!userId) {
        res.status(403).send('Unauthorized. You shall not hack!');
    }

    const user = UserHandler.getUserDetails(userId);
    const secrets = SecretsHandler.getSecretsByUser(user);

    return res.status(200).send({
        user,
        secrets,
    });
});
```

{% hint style="danger" %}
Secure all methods that each path has! It is quite common for a developer to protect the GET path on a specific endpoint, however, the DELETE/ POST/ PUT ones would still be vulnerable.
{% endhint %}

### 3.2. Working without access control:

Although not recommended, here you have a **"security through obscurity"** mitigation. You can reference  objects with random and unpredictable values instead of simple numeric IDs. However, simply using a random ID cannot be considered excellent protection since IDs can be leaked or stolen, even though they are hard to guess/ craft. Example:

`/api/e48e13207341b6bffb7fb1622282247b/secrets/`

## 4. Takeaways:

Implementing a proper access control policy + an impeccable session management should rid you of most **Broken Object Level Authorization** issues. 

{% hint style="info" %}
You can find more details about this topic here:

* [OWASP API SECURITY](https://owasp.org/www-project-api-security/)
* [What is Broken Object Level Authorization ](https://www.wallarm.com/what/broken-object-level-authorization)
{% endhint %}

