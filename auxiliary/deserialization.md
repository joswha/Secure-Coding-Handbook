# Deserialization

## 1. Introduction:

 **Serialization** is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes. Serializing data makes it much simpler to:

*  Write complex data to inter-process memory, a file, or a database.
*  Send complex data, for example, over a network, between different components of an application, or in an API call.

Conversely,  **deserialization** is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized. The website's logic can then interact with this deserialized object, just like it would with any other object.

```php
// Serializing an object
<?php

class User {
    public $username;
    public $role;
    public $email;
}

$user = new User;
$user->username = 'bobi';
$user->role = 'admin';
$user->email = 'bobi@google.com';

echo serialize($user);

// Output 
// O:4:"User":3:{s:8:"username";s:4:"bobi";s:4:"role";s:5:"admin";s:5:"email";s:15:"bobi@google.com";}
?>
```

Many programming languages offer default capabilities\(or formats\) of serializing objects. These native **seralization** formats typically come with more features\(eg. _customizability\)_ than the ever-so-used **JSON and XML formats**. However, this "room" for customizability allows for a open window in the application's security; attackers can leverage the **deserialization** process to their advantage. You can read more about **exploiting deserialization**, [from an attacker's perspective, here](https://portswigger.net/web-security/deserialization/exploiting).

## 2. Typical vulnerable application:

 **Deserialization** typically arises because there is a general lack of understanding of how **dangerous** deserializing **user-controllable data** can be. Ideally, **user input should never be deserialized at all.** Take, for example, [this code snippet from OWASP](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection).

```php
<?php

class Example2
{
   private $hook;

   function __construct()
   {
      // ...
   }

   function __wakeup()
   {
      if (isset($this->hook)) eval($this->hook);
   }
}

// ...

$user_data = unserialize($_COOKIE['data']);

// ...
?>
```

On line 14 we clearly observe the fact that the application runs `eval` on user-supplied input. All that an attacker has to do in order to exploit this is to craft a `data` cookie like this:

```php
class Example2
{
   private $hook = "phpinfo();"; // phpinfo() is the `injected` code.
}

// We have to urlencode the output due to sending it via HTTP request
print urlencode(serialize(new Example2));
```

This is the HTTP request that triggers the actual exploit. 

```http
GET /vuln.php HTTP/1.0
Host: testsite.com
Cookie: data=O%3A8%3A%22Example2%22%3A1%3A%7Bs%3A14%3A%22%00Example2%00hook%22%3Bs%3A10%3A%22phpinfo%28%29%3B%22%3B%7D
Connection: close
```

Note that we basically:

1. Craft the serialized **data** cookie.
2. Application calls **unserialize** our crafted **data** cookie.
3. **\_wakeup\(\)** is called by the **unserialized\(\)**. It looks for the **$hook** and runs **eval\($hook\)**
4. **eval\("phpinfo\(\);"\)** is run. CHAOS!

Although sometimes developers think they are safe because they implement some form of additional check on the deserialized data, this approach is often ineffective. Since in most cases trying to verify/ validate or check anything is only possible **AFTER** the deserialization has happened, it is already too late to do anything, your application is screwed!

## 3. Mitigations:

 Generally speaking, **deserialization** of **user input should be avoided unless absolutely necessary**. The high severity of exploits that it potentially enables, and the difficulty in protecting against them, outweigh the benefits in many cases.

 If you do need to **deserialize data from untrusted sources**, incorporate robust measures to make sure that the **data has not been tampered with**. For example, you could implement a digital signature to check the integrity of the data. However, remember that any checks must take place **before** beginning the deserialization process. Instead, you could create your own **class-specific serialization methods** so that you can at least control which fields are exposed.

[OWASP recommends the usage](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html#whitebox-review) of a safer interchange format, such as **JSON**, using `json_decode()` in the absolute necessary case of passing some sort of serialized data to the user.  

## 4. Takeaways:

Finally, remember that the vulnerability is the **deserialization** of user input, **not the presence of gadget chains that subsequently handle the data**. Don't rely on trying to eliminate gadget chains that you identify during testing. It is impractical to try and plug them all due to the web of cross-library dependencies that almost certainly exist on your application. At any given time, publicly documented memory corruption exploits are also a factor, meaning that your application may be vulnerable regardless.

{% hint style="info" %}
You can find more details about this topic here:

* [Insecure Deserialization](https://portswigger.net/web-security/deserialization)
* [Deserialization \[OWASP\]](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
* [PHP Object Injection](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)
* [Never Pass Untrusted Data to Unserialize](https://www.netsparker.com/blog/web-security/untrusted-data-unserialize-php/)
{% endhint %}

