# Mass Assignment

## 1. Introduction:

Modern frameworks encourage developers to use functions that automatically bind input from the client into code variables and internal objects. \(Easier to code  !-&gt; safe code\) The binding of client-provided data \(e.g., JSON\) to data models, without adequate properties filtering based on an allowlist, usually leads to Mass Assignment. Either through guessing objects properties\(eg. `admin: true`, `role: admin`\), exploring other API endpoints, reading the documentation, or providing additional object properties in request payloads, allows attackers to modify object properties they are not supposed to. 

You can read more about Mass Assignment, from an attacker's perspective, [here](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html).

## 2. Typical vulnerable code:

In the usual vulnerable scenario, the developer does not bind what aspect of the object they want to update through the request. Take a look at the following code:

```javascript
const express = require('express');
const app = express();
const router = app.router;

const UserManager = require('UserManager');

/*

Imagine that the schema of the User object looks like:

{
    "uid": String,
    "email": String,
    "password": String,
    "username": String,
    "role": String
}

*/

router.post('/api/reset-password', (req, res) => {

    const token = req.getHeader('Authorization');
    const isValid = UserManager.validateToken(token);
    
    if (isValid) {
        const user = req.body.user;
        UserManager.updateUserSettings(user);
        res.status(200).send();
    } else {
        res.status(403).send();
    }
    
});
```

And now assume that the request would look similar to this:

```http
POST /api/reset-password

.....

{
    "user": {
        "password": "new_password"
    }
}
```

Since the developer does not specifically bind which fields of the accounts they want to update, it follows suit that we can input ANY field\(as long as we know/ guess them\) and hence update all those fields. With the request of:

```text
POST /api/reset-password

.....

{
    "user": {
        "password": "new_password",
        "role": "admin"
    }
}
```

An attacker successfully updates their `role` field into `admin`, thus escalating their privileges. 

{% hint style="warning" %}
This is just a typical attack scenario, and one can infer that there are several other ways of exploiting this vulnerability\(eg. changing another user's password/ email/ username/ etc\)
{% endhint %}

## 3. Mitigations:

By far the safest method of protecting against such a vulnerability is using a schema database validation framework\(such as [mongoose](https://mongoosejs.com/docs/guide.html)\). You can also specifically retrieve the object that you want from the request\(eg. `req.body.password`\) and have a function that would **ONLY** update the `password` field. Here's how to patch the code:

```javascript
const express = require('express');
const app = express();
const router = app.router;

var UserSchema = new mongoose.Schema({
    uid: String,
    email: String,
    password: String,
    username: String,
    role: String
});

const User = mongoose.model('User', UserSchema);
​
router.post('/api/reset-password', (req, res) => {

    const user_id = req.session.user_id;
    
    if (user_id) {

        User.findbyId(user_id, function (err, user) {
            //   ...
              user.password = req.body.new_password;
              user.save();
            //   ...
            }
        });

    } else {
        res.status(403).send();
    }

});
```

Note that on line `23` we specify which exact field of the user object we want to update.

## 4. Takeaways:

Following these essential points will defend your endpoints against Mass Assignment:

* Bind the fields that you want to work with from the request.
* Do not blindly trust the methods of the frameworks you are working with. Read the documentation!
* All schemas and types that are expected in the requests should be explicitly defined at design time and enforced at runtime.

{% hint style="info" %}
You can find more details about this topic here

* [CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes](https://cwe.mitre.org/data/definitions/915.html)
* [Mass Assignment](https://raw.githubusercontent.com/OWASP/API-Security/master/2019/en/dist/owasp-api-security-top-10.pdf)
* [Mass Assignment Application Security](https://application.security/free-application-security-training/owasp-top-10-api-mass-assignment)
{% endhint %}

