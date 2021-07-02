# XML External Entity Injection \[XXE\]

## 1. Introduction:

This attack occurs when untrusted XML input containing a **reference to an external entity is processed by a weakly configured XML parser**.

It may lead to the disclosure of confidential data, denial of service, [Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) \(SSRF\), port scanning from the perspective of the machine where the parser is located, and other system impacts. 

## 2. Typical vulnerable code: 

{% tabs %}
{% tab title="Javascript" %}
```javascript
var parserOptions = {
    noblanks: true,
    noent: true,
    nocdata: true
};

try {
    var doc = libxml.parseXmlString(data, parserOptions);
} catch (e) {
    return Promise.reject('XML parse error');
}
```
{% endtab %}

{% tab title="Python" %}
```python
from django.http import HttpResponse
from lxml import etree

def authenticate(content):
    parser = etree.XMLParser(resolve_entities=True)
    try:
        document = etree.fromstring(content, parser)
    except etree.XMLSyntaxError:
        return None
```
{% endtab %}

{% tab title="PHP" %}
```php
class Loader
{
    /**
    * @param string $path
    * @return DOMDocument
    */
    public function load($path)
    {
         $dom = new DOMDocument();
         $dom->loadXML(file_get_contents($path));
         return $dom;
    }
}
```
{% endtab %}
{% endtabs %}

An XXE attack works by taking advantage of a feature in XML, namely **XML** e**X**ternal **Entities** \(XXE\) that allows external XML resources to be loaded within an XML document.  
  
 By submitting an XML file that defines an external entity with a `file://` URI, an attacker can effectively trick the **application's SAX\(Simple API parser for XML\) parser** into reading the contents of the arbitrary file\(s\) that reside on the server-side filesystem.

Finally, we use the XML declaration `ENTITY` to load additional data from an external resource. The syntax for the `ENTITY` declaration is `ENTITY name SYSTEM URI` where `URI` is the full path to a remote URL or local file. In our example, we define the `ENTITY` tag to load the contents of `"file:///etc/passwd"`

This is an example of how the XML "payload" file can look like. 

{% code title="payload.xml" %}
```markup
<!DOCTYPE foo [<!ELEMENT foo ANY >
<!ENTITY bar SYSTEM "file:///etc/passwd" >]>
<?xml version="1.0" encoding="UTF-8"?>

<trades>
    <metadata>
        <name>Apple Inc</name>
        <stock>AAPL</stock>
        <trader>
            <foo>&bar;</foo>
            <name>B.Bobi</name>
        </trader>
        <units>1500</units>
        <price>130</price>
        <name>Google</name>
        <stock>GOOGL</stock>
        <trader>
            <name>B.Bobi</name>
        </trader>
        <units>1500</units>
        <price>130</price>
    </metadata>
</trades>
```
{% endcode %}

When this XML document is processed by a vulnerable parser, such as the one presented above, any instances of `&bar;` will get replaced by the contents of `/etc/passwd` file.  
  
 Simply put **the XML parser was tricked into accessing a resource** that the application developers **did not intend to be accessible**, in this case, **a file on the local file system of the remote server**. Because of this vulnerability, **any file on the remote server** \(or more precisely, any file that the web server has read access to\) could be obtained.

Unfortunately, the **SAX parser has not been configured to deny the loading** of external entities \( `DOCTYPE` declarations \), which when specified within our modified `payload.xml` file can be **abused by an attacker to read arbitrary system files** on the remote server.

{% hint style="info" %}
Because user-supplied XML input comes from an "untrusted source"\(namely, the client\) **it is very difficult to properly validate the XML document** to **prevent this type of attack**.
{% endhint %}

## 3. Mitigation:

### 2.1. Disabling inline DTD:

Since **we know that the parser should technically not allow loading external entities**\( any kind of `DOCTYPE` declaration\), we can **have the XML parser configured to only use a locally defined Document Type Definition \(DTD\)** and **disallow any inline DTD** that is specified within a user-supplied XML document\(s\).  
  
 Due to the fact that there are **numerous XML parsing engines available,** each has its own mechanism for disabling inline DTD to prevent XXE. You may need to search your XML parser's documentation for how to "disable inline DTD" specifically.

Here are some examples of how you may disable the inline **DTD parsing**.

{% tabs %}
{% tab title="Javascript" %}
```javascript
var parserOptions = {
    noblanks: true,
    noent: false, // doesn't allow DOCTYPE declarations
    nocdata: true
};

try {
    var doc = libxml.parseXmlString(data, parserOptions);
} catch (e) {
    return Promise.reject('XML parse error');
}
```
{% endtab %}

{% tab title="Python" %}
```python
from django.http import HttpResponse
from lxml import etree

def authenticate(content):
    parser = etree.XMLParser(resolve_entities=False)
    # False -> doesn't allow DOCTYPE declarations
    try:
        document = etree.fromstring(content, parser)
    except etree.XMLSyntaxError:
        return None
```
{% endtab %}

{% tab title="PHP" %}
```php
class Loader
{
    /**
    * @param string $path
    * @return DOMDocument
    */
    public function load($path)
    {
        $old = libxml_disable_entity_loader(true);
        // disable entity loader -> does not alow DOCTYPE declarations
        $dom = new DOMDocument();
        $dom->loadXML(file_get_contents($path));
        libxml_disable_entity_loader($old);
        return $dom;
    }
}
```
{% endtab %}
{% endtabs %}

### 2.2. Follow the principle of least privilege:

The threat of **XXE** attacks entirely illustrates the importance of **following the principle of least privilege**, which states that **software components and processes should be granted the minimal set of permissions** required to perform their tasks. 

Since there are rarely good reasons for an XML parser to make outbound network request, consider locking down outbound network requests for your web server as a whole. If you do need outbound network access: for example, if your server code calls third-party APIs—you should **whitelist the domains of those APIs in your firewall rules**.

## 3. Conclusions:

It is common knowledge that DTDs are legacy technology, thus allowing for **inline DTDs** is **ALWAYS A BAD IDEA!** Modern XML parsers are hardened by default, and because of this, using such frameworks or parsers means that you might already be protected against attacks.

{% hint style="info" %}
You can find more details about this topic here:

* [XML External Entity Prevention Cheatsheet \[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
* [What is XXE? ](https://portswigger.net/web-security/xxe)
{% endhint %}

