# Cross-Site Request Forgery \[CSRF\]

## 1. Introduction:

Cross-site request forgery \(also known as **CSRF**\) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same-origin policy, which is designed to prevent different websites from interfering with each other. In the usual attack scenario, a **`GET`** request that changes the state of the webserver is exploited. 

You can read more about this type of vulnerability, from the attacker's perspective, [here](https://portswigger.net/web-security/csrf).

## 2. Defensive methods:

### 2.1. Following the REST architecture.

**REST** or **Representational State Transfer** states that **`GET`** requests should strictly be used when fetching data or other resources and that for any other actions that would essentially change the server state, you **should** use one of the appropriate protocols, such as **`PUT`**, **`POST`** and **`DELETE`**.

Since not all actions have an obvious corresponding HTTP method, such as fetching = **`GET`**, **updating** = **`POST`**, **creating** = **`PUT`**, **delete** = **`DELETE`**, there are other mitigations that can secure our application as much as possible, though for now, it is important to remember that **`GET`** should strictly be used for fetching data.

### 2.2. Working with CSRF tokens.

This is one of the most recommended methods to properly mitigate CSRF. 

{% hint style="info" %}
CSRF tokens prevent CSRF because, without them, an attacker cannot create any valid requests to the backend server.
{% endhint %}

Listed below are a few techniques and use cases for those tokens.

#### 2.2.1. Synchronizer token.

Ideally, a developer would store CSRF tokens on the server-side. Depending on the implementation, those can either be **per-session** or **per-request** tokens. **Per-request** tokens are typically more secure than the **per-session** one, as the time span that they are valid is rather limited; this comes with usability limitations, however, using the "back" browser button for example would not work since the session would have expired already**.**

Upon a request issue by the client, the server-side component should have checked the existence and validity of the token in the request compared to the token in the session. Should there either be no token at all or the request token mismatches the value in the session, the request should be aborted and even marked as a potential CSRF attack.

Upon designing the CSRF tokens, a developer should know that tokens have to:

* Be **unique** per user session.
* **Secret** 
* **Unpredictable** \(typically done through a secure generating method\).

{% hint style="danger" %}
**CSRF** **tokens** should **NOT** be transmitted through cookies.
{% endhint %}

**CSRF** tokens can be added through hidden fields, headers, and can be used with forms, and AJAX calls. It is important to check for any possible leakages, such as the server logs or even in the URL. Here you have an example of applying the token to a form.

```markup
<form action="/pay" method="post">
    <input type="hidden" name="CSRFtoken" value="ZjJkMzA3YWEyMzg2YTBjNzQzM2NlODUwMTllZTU2MTk=">
    [...]
</form>
```

#### 2.2.2. Double submit cookies.

Maintaining a state for the CSRF tokens is sometimes problematic, and thus an alternative for the synchronizer token is the double submit cookie technique. 

When a user visits\(preferably before authentication\) the website should \(securely generate\) a value and set that as a cookie on the user's browser. The application thus requires that any action request includes this value as a hidden **form** value. Thus, should both the hidden form value and this cookie match on the server-side, the server accepts the request as legitimate, otherwise, it rejects it.

{% hint style="info" %}
Note that this method is somewhat of a workaround and should preferably be used together with some way of encrypting those cookies, or working with[ **HMAC**](https://www.nedmcclain.com/better-csrf-protection/).
{% endhint %}

### 2.3. Use of custom request headers.

Adding CSRF tokens, using the double submit cookie or other defense that involves changing the UI can frequently be complex or otherwise problematic. An alternative defense method that is particularly well suited for AJAX or API endpoints is the use of a **custom request header**. This relies on the [same-origin policy \(SOP\)](https://en.wikipedia.org/wiki/Same-origin_policy), which states that JavaScript can be used for adding a custom header, limited strictly within its origin. By default, however, browsers do not allow JavaScript to make any cross-origin requests with custom headers.

Because the **`POST`**, **`PUT`**, **`PATCH`**, and **`DELETE`** are state-changing methods, they should use a CSRF token attached to the request. The following guidance will demonstrate how to create overrides in JavaScript libraries to have CSRF tokens included automatically with every AJAX request for the state-changing methods mentioned above.

{% tabs %}
{% tab title="Axios" %}
```markup
<script type="text/javascript">
    var csrftoken = document.querySelector("meta[name='csrf-token']").getAttribute("content");

    axios.defaults.headers.post['anti-csrf-token'] = csrftoken;
    axios.defaults.headers.put['anti-csrf-token'] = csrftoken;
    axios.defaults.headers.delete['anti-csrf-token'] = csrftoken;
    axios.defaults.headers.patch['anti-csrf-token'] = csrftoken;

</script>
```
{% endtab %}
{% endtabs %}

### 2.4. Using a CSRF protection middleware.

In the end, using a "battle-tested" method such as a module is one of the safest defensives. As a proof of concept, I will showcase how using a [csurf](https://www.npmjs.com/package/csurf) will help you generate and manage CSRF tokens. 

{% tabs %}
{% tab title="NodeJS" %}
```javascript
// server.js
var cookieParser = require('cookie-parser')
var csrf = require('csurf')
var bodyParser = require('body-parser')
var express = require('express')
 
// setup route middlewares
var csrfProtection = csrf({ cookie: true })
var parseForm = bodyParser.urlencoded({ extended: false })
 
// create express app
var app = express()
 
// parse cookies
// we need this because "cookie" is true in csrfProtection
app.use(cookieParser())
 
app.get('/form', csrfProtection, function (req, res) {
  // pass the csrfToken to the view
  res.render('send', { csrfToken: req.csrfToken() })
})
 
app.post('/process', parseForm, csrfProtection, function (req, res) {
  res.send('data is being processed')
})
```
{% endtab %}
{% endtabs %}

And now, set the csrfToken as the value of the hidden input field called `_csrf`.

{% tabs %}
{% tab title="ReactJS" %}
```markup
<form action="/process" method="POST">
  <input type="hidden" name="_csrf" value="{{csrfToken}}">
  
  Favorite color: <input type="text" name="favoriteColor">
  <button type="submit">Submit</button>
</form>
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
You can find more details about this topic here:

* [Should I use CSRF protection on Rest API endpoints?](https://security.stackexchange.com/questions/166724/should-i-use-csrf-protection-on-rest-api-endpoints/166798#166798)
* [CSRF Protection Cheatsheet \[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#javascript-guidance-for-auto-inclusion-of-csrf-tokens-as-an-ajax-request-header)
* [Preventing CSRF](https://auth0.com/blog/cross-site-request-forgery-csrf/)
{% endhint %}



