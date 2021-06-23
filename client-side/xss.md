# Cross-Site Scripting \[XSS\]

## 1. Introduction:

This page highlights the basic principles of defending against XSS, regardless of the attack vector. We will, however, not dig into the negative repercussions that XSS attacks could cause; for more information regarding further exploitations of this attack, [please take your time reading this](https://owasp.org/www-community/attacks/xss/).

Even if your server is secure, any hacker's best target is the web browser. Quite frankly, the browsers execute any JavaScript code that appears on any web page. Since the cross-site scripting attack is a really common one, we can divide it into three types: 

* **Stored XSS.**
* **Reflected XSS.**
* **DOM-Based XSS.**

## 2.1. Stored Cross-Site Scripting:

**Stored attacks** are those where the injected script is permanently stored on the target servers, such as in a database, in a message forum, visitor log, comment field, etc. The victim then is targeted by the malicious script from the server when it requests its stored information. **Stored XSS** is also sometimes referred to as **Persistent** or **Type-I XSS**.

### 2.1.1. Escaping HTML Characters:

The first step towards preventing **stored cross-site scripting** ideally means escaping all dynamic content coming from a database, such that the browser interprets **the content** of HTML tags, instead of interpreting the entire content as raw HTML.

| **Character** | **Entity encoding** |
| :--- | :--- |
| & | **&amp;** |
| **'** | **&apos;** |
| **&lt;** | **&lt;** |
| **&gt;** | **&gt;** |

As so, an example of how escaping and displaying the retrieved information from the database could look like:

```markup
<div class="message">
    <h1> Hello, this is the message:
        &lt;script&gt;alert(&quot;Hey&quot;)&lt;/script&gt;
    </h1>
</div>
```

This conversion of escaped characters happens, of course, **after** the browser has constructed the DOM for the page, such that it will **not** execute the `<script>` tag. Since cross-site scripting is such a **common** **vulnerability**, modern **front-end frameworks** more likely than not **already escape dynamic content** by default. Usually, **string** variables in views are escaped automatically. 

Here is an example of how **ReactJS** deals with escaping the response:

```jsx
const message = "<script>alert('Hey')</script>"

class UserProfilePage extends React.Component {
  render() {
    return (
        <div class="message">
          <h1> Hello, this is the message: {message}!</h1>
        </div>
    );
  }
}
```

{% hint style="danger" %}
Although **front-end frameworks tend to already escape dynamic content,** this is only limited to actually **displaying it**. If there is a case of using that content within `<a href={...} />, <img src={...} />` the developer should **take other defensive measures** of making sure that the retrieved data is properly escaped.
{% endhint %}

### 2.1.2. Implementing a Content Security Policy \[CSP\]:

We will dedicate an entire page for everything related to CSP,  however, it is worth mentioning some general aspects of how CSPs protect against cross-site scripting.

Modern browsers allow websites to set a content security policy, which you can use to **lock down JavaScript execution** on your site. 

A very basic policy that limits the imported scripts of a page to the **same domain**\(`self`\), and tells the browser **that inline JavaScript** should **NOT** be executed.

```markup
Content-Security-Policy: script-src 'self' https://scripts.github.com
```

You can also set your site’s content security policy in a `<head>` tag in the HTML of your web pages.

## 2.2. Reflected Cross-Site Scripting:

**Reflected attacks** are those where the injected script is reflected off the web server, such as in an error message, search results, or any other response that includes some or all of the user input part of the request. 

When a victim is tricked into **clicking on a malicious link**, submitting a specially crafted form, or even just browsing a malicious site, the injected code "travels" to the vulnerable website, which **reflects** the attack back to the user’s browser. The browser then executes the code because it came from a “trusted” server. **Reflected XSS** is also sometimes referred to as **Non-Persistent** or **Type-II XSS**.

### 2.2.1. Escapic Dynamic Content from the HTTP Requests:

This mitigation closely follows the one discussed at **2.1.1.** Whether the dynamic content comes from the backend/ database or the HTTP request itself, it is **escaped in the same way**. Now luckily, modern front-end templates escape all variables, regarding where they came from\(HTTP request or backend\). 

{% hint style="warning" %}
Common target areas for reflected XSS are search pages and error pages since they display parts of the **query string back to the user.**
{% endhint %}

## 2.3. Document Object Model \[DOM\]-Based Cross-Site Scripting:

 **DOM-based XSS vulnerabilities** usually arise when **JavaScript** takes data from an attacker-controllable source, such as the URL, and passes it to a sink that supports dynamic code execution, such as `eval()` or `innerHTML`. This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts.

**Reflected** and **Stored** **XSS** are _server-side_ injection issues while **DOM-based XSS** is a _client \(browser\) side_ injection issue. With **Reflected/Stored** the attack is injected into the application during **server-side** processing of requests where untrusted input is **dynamically** added to HTML. For **DOM XSS**, the attack is injected into the application during runtime in the **client** directly.

### 2.3.1. Vulnerable code example:

Here you have an example of how a vulnerable page could look like, using HTML5 and JavaScript:

```javascript
function refreshItems() {
    
    const type = (new URL(location.href))
        .searchParams.get('filter')
        .replace('+', ' ');

    const activeItemLink = document.querySelector(`.itemlink[data-type=${type}]`);

    if(activeItemLink) {
        activeTab.classList.add('active');
    }

    // Search items
    const items = type ? data.filter(item => {
        return item.type === type;
    }) : data;

    // Show current type name
    document.getElementById('currentItemName').innerHTML = type; 
    // !!!! no input validation before appending the value to DOM
    // here we basically append the type value to the ODM by passing it 
    // to the innerHTML of the current item.
    
    // Display items
    let itemsHTML = '';

    // To render and update each active item,
    // it is extracted from the URL query parameter filter
    items.forEach(item => {
        itemsHTML == 
        `
            <div>
                <img src="${item.icon}">
                <p>${item.name}</p>
                <p>${item.description}</p>
                <p>${item.owner}</p>
            </div>
        `;
    });

    document.getElementById('list').innerHTML = itemsHTML;
}

document.addEventListener('DOMContentLoaded', () => {
    const itemLinks = document.getElementsByClassName('itemlink');

    itemLinks.foreach(link => {
        link.addEventListener('click', (event) => {
            location.search = `?filter=${event.target.innerText}`;
        });
    });

    refreshItems();
})
```

As you can tell from this line:

```javascript
document.getElementById('currentItemName').innerHTML = type; 
```

We are appending the value of the current item's name to the DOM without any input validation, and this is vulnerable to **DOM XSS** injection, such that should an attacker input a malicious injection, our item's listing will execute the injection.

### 2.3.2. Mitigation:

We will follow suit with the **Stored XSS** and **Reflected XSS** defensive techniques, in the sense that first things first we have to **escape** all user input. Since our code is plain JavaScript, we have to create a new function that does that for us:

```javascript
function escapeHTML(html) {
    return html
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt')
        .replace(/</g, '&quot;')
        .replace(/'/g, '&#39;');
}
```

Now, we have to escape what is going to be displayed, and that is our user-inputted query. As an additional layer of security, we can use `textContext` instead of `innerHTML` since we do not want to change the HTML node itself anyway. Apart from this, `textContext` also escapes the HTML markup characters so we are as escaped, thus preventing the malicious HTML from being executed.

```javascript
document.getElementById('currentItemName').textContext = escapeHTML(type); 
```

{% hint style="info" %}
You can find more details about this topic here:

* [Stored XSS Example](https://application.security/free-application-security-training/owasp-top-10-stored-cross-site-scripting)
* [Reflected XSS Example](https://application.security/free-application-security-training/owasp-top-10-reflected-cross-site-scripting)
* [DOM XSS Example](https://application.security/free-application-security-training/owasp-top-10-dom-cross-site-scripting)
* [XSS Prevention Cheatsheet\[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
* [DOM XSS Prevention Cheatsheet\[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
* [What is DOM-based XSS?](https://portswigger.net/web-security/cross-site-scripting/dom-based)
{% endhint %}

