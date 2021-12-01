# Template Injection \[SSTI]

## 1. Introduction:

&#x20;**Server-side template injection** is when an attacker is able to use **native template syntax** to inject a malicious payload into a template, which is then **executed server-side**.

&#x20;Template engines are designed to **generate web pages** by combining **fixed templates** with **volatile data**. This allows attackers to **inject arbitrary template directives** in order to **manipulate the template engine**, often **enabling them to take complete control of the serve**r.&#x20;

You can read more about this type of vulnerability, [from an attacker's perspective, here.](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)

## 2. Typical vulnerable code:

There are several libraries/ frameworks that make use of templates, here are a few: **Jinja, Flask, Mako, Twig. The following code snippet showcases Flask's vulnerability:**

```python
from flask import Flask,request,render_template_string 
from urllib.parse import unquote

app = Flask(__name__)

@app.route("/")
def main_page():
    return "Hey there, this is my cool weeb site."

@app.errorhandler(404) 
def page_not_found(error): 
    url = unquote(request.url) 
    return render_template_string("<h1>URL %s not found</h1><br/>" % url), 404 
    
if __name__ == '__main__': 
    app.run(debug = False, host = '0.0.0.0')
```

![](<../.gitbook/assets/image (7).png>)

The given input is being **rendered and reflected** into the response. This is easily **mistaken for a simple** [**XSS**](https://vladtoie.gitbook.io/secure-coding/client-side/xss) vulnerability, but it's easy to difference if you try set **mathematical operations** within the template expression: `{{7*7}}`.

![](<../.gitbook/assets/image (8).png>)

Showcasing that `{{7*7}}` gets rendered as `49` proves the point that our application is vulnerable to SSTI. We will however not go into more details regarding further exploitation, [however you can refer to this awesome guide](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection) for that.

## 3. Mitigations:

If **user-supplied templates are a business requirement**, how should they be implemented? The lowest risk approach is to simply use a trivial template engine such as Mustache, or Python's Template. **Separating the logic from rendering** as much as possible can greatly reduce your exposure to the most dangerous template-based attacks. Another, complementary approach is to **concede that arbitrary code execution is inevitable**(regardless of **filtering/ whitelisting/ blacklisting)**and sandbox it inside a locked-down Docker container.&#x20;

## 4. Takeaways:

Not using user-supplied templates saves you of this possible exploitation. Should you really have to use template rendering however, an alternative can be sandboxing the actual rendering in a custom way, though you can think of the major drawbacks here.&#x20;

{% hint style="info" %}
You can find more details about this topic here:

* [Server-Side Template Injection](https://portswigger.net/web-security/server-side-template-injection).
* [Hacktricks guide of exploiting SSTI.](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
* [Testing for SSTI](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web\_Application\_Security\_Testing/07-Input\_Validation\_Testing/18-Testing\_for\_Server\_Side\_Template\_Injection)
{% endhint %}

{% file src="../.gitbook/assets/EN-Server-Side-Template-Injection-RCE-For-The-Modern-Web-App-BlackHat-15 (1).pdf" %}
BlackHat SSTI
{% endfile %}
