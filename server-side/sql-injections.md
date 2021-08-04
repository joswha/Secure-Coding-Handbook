# SQL Injections \[SQLi\]

## 1. Introduction: 

SQL Injections happen when developers create **dynamic database queries** that include **user-supplied input**. For this reason, developers have to:

1. Stop writing **dynamic queries** for their applications.
2. **Filter and escape the user-supplied input**.

The techniques presented here technically apply to most other programming languages and/ or types of databases.  

## 2. Typical vulnerable code:

{% tabs %}
{% tab title="JavaScript" %}
```javascript
function authenticate(req, res, next) {
    var email = req.query.email,
        password = req.query.password,
        sqlRequest = new sql.Request(),
        sqlQuery = "select * from users where (email='" 
        + "' and password = '" + password + "')";
    
    sqlRequest.query(sqlQuery)
        .then(function (recordset) {
            if (recordset.length == 1) {
                loggedIn = true;
                // Auth successful
            } else {
                // Auth failure
            }
        })
        .catch(next);
}
```
{% endtab %}

{% tab title="Python" %}
```python
def authenticate(request):
    email = request.POST['email']
    password = request.POST['password']
    sql = "select * from users where (email ='" 
        + email 
        + "' and password ='" + password + "')"
    
    cursor = connection.cursor()
    cursor.execute(sql)
    row = cursor.fetchone()
    if row:
        loggedIn = "Auth successful"
    else:
        loggedIn = "Auth failure"
    return HttpResponse("Logged In Status: " + loggedIn)
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
$email = $POST['email'];
$password = $POST['password'];

$stmt = mysql_query("SELECT * FROM users WHERE (email='$email' 
                    AND password='$password') LIMIT 0,1"
                    );

$count = mysql_fetch_array($stmt);
if($count > 0) {
    session_start();
    // Auth successful
}
else
{
    // Auth failure
}
?>
```
{% endtab %}
{% endtabs %}

**`sqlQuery`** executes an **SQL Query** without any input validation whatsoever. \(i.e. no checking of legal characters, minimum/maximum string lengths, or removal of "malicious" characters\).

The attacker can inject raw **SQL syntax** within both the username and the password input fields to alter the meaning of the underlying **SQL query** responsible for authentication, resulting in a bypass of the application's authentication mechanism. An example of such bypass could be the query: `' or 1=1)#`

## **3. Mitigation:**

[SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) attacks are **unfortunately very common**, and this is due to two factors:

1. The **significant prevalence of SQL Injection** vulnerabilities, and
2. The **attractiveness of the targe**t \(i.e., the **database typically contains all the interesting/critical** data for your application\).

It's rather shameful that there still are so many **successful SQLi** occurring because it is really simple to avoid this type of vulnerability in the first place. In this article, you will discover the **top 4 most reliable techniques** for developing a **robust** and **secure** **application**.

### **3.1. Prepared statements:**

The use of **prepared statements with variable binding** \(aka **parameterized queries**\) is how all developers should write their queries to begin with. They are **simple to write** and **easier to understand than dynamic queries**. Parameterized queries **force the developer to first define all the SQL code**, and then **pass in each parameter to the query later**. This coding style allows the database to **distinguish between code and data**, **regardless** of what user input is **supplied**.

Prepared statements **ensure that an attacker is not able to change the intent of a query**, even if SQL commands are inserted by an attacker. In the safe example below, if an attacker were to enter the email of `bobi@mail.com' or '1'='1`, the **parameterized query** would **not be vulnerable** and would instead look **for a username that literally matched the entire string** `bobi@mail.com' or '1'='1`.

{% tabs %}
{% tab title="JavaScript" %}
```javascript
function authenticate(req, res, next) {
    var email = req.query.email,
        password = req.query.password,
        ps = new sql.PreparedStatement(),
        sqlQuery = "select * from users where (email = @email and " 
                 + "password = @password)";
    
    ps.input('email', sql.VarChar(50));
    ps.input('password', sql.VarChar(50));
    ps.prepare(sqlQuery)
    .then(function () {
        return ps.execute({email: email, password: password})
        then(function (recordset) {
            if (recordset.length == 1) {
                loggedIn = true;
                // Auth successful
    
            } else {
                // Auth failure
            }
        })
    })
    .catch(next);
};
```
{% endtab %}

{% tab title="Python" %}
```python
def authenticate(request):
    email = request.POST['email']
    password = request.POST['password']
    
    cursor = connection.cursor()
    cursor.execute("select * from users where(email = %s and password = %s)"
    , [email, password])
    row = cursor.fetchone()
    if row:
        loggedIn = "Auth successful"
    else:
        loggedIn = "Auth failure"
    return HttpResponse("Logged In Status: " + loggedIn)
```
{% endtab %}

{% tab title="PHP" %}
```php
<?php
$email = $POST['email'];
$password = $POST['password'];

$stmt = $db->prepare("SELECT * FROM users WHERE 
                    (email=:email AND password=:password) LIMIT 0,1"
                    );
$stmt->bindParam(':email', $email);
$stmt->bindParam(':password', $password);
$stmt->execute();

$count = mysql_fetch_array($stmt);
if($count > 0) {
    session_start();
    // Auth successful
}
else
{
    // Auth failure
}
?>
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Wherever **executing SQL queries** is necessary, make sure **to always use** [prepared statements and query parametrization](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html).
{% endhint %}

### 3.2. Stored procedures:

**Stored procedures are not always safe** from SQLi. However, **certain standard stored procedure** programming constructs **have the same effect as the use of parameterized queries** when **implemented safely**.

They require the developer to just **build SQL statements with parameters that are automatically parameterized** unless the developer does something largely out of the norm. The difference between prepared statements and stored procedures is that the **SQL code for a stored procedure is defined and stored** in the **database itself**, and then **called from the application**. 

Both of these techniques have t**he same effectiveness in preventing SQL injection** so your organization should choose which approach makes the most sense for you.

This is a short example of how a stored procedure can be implemented, in SQL.

```sql
CREATE PROCEDURE SafeAuth(@username varchar(50),  @password varchar(50))
AS
BEGIN
DECLARE @sql varchar(150)
    SELECT Username,Password FROM dbo.Login
    WHERE Userame=@username AND Password=@password
end
```

**Note**: **'Implemented safely'** means the **stored procedure does not include** any unsafe dynamic SQL generation. Developers do not usually generate dynamic SQL inside stored procedures. However, it can be done but should be avoided. **If it can't be avoided**, the stored procedure **must use input validation** or **proper escaping** as described in this article to make sure that **all user-supplied inpu**t to the stored procedure can't be used to inject SQL code into the dynamically generated query. 

{% hint style="success" %}
Although stored procedures are an efficient way to safe proof against SQLI, it is crucial to also combine them with another technique, such as **prepared statements**.
{% endhint %}

### 3.3. Input validation:

There are two types of input validation: **syntactical** and **semantical.**

**Syntactical** validation enforces syntax correctness of _structured fields_. \(eg. SSN, date, currency symbol, etc\).

**Semantic** validation imposes the correctness of the inputted _values_ in the specific **business context**. \(eg. start date before the end date, price within attributed range, etc\)

I**nput validation** can be implemented using any programming technique that allows effective enforcement of syntactic and semantic correctness, for example:

{% hint style="danger" %}
It is **important** to note however that any JavaScript **input validation** performed on the **client-side** can be bypassed by an attacker that disables JavaScript or uses a Web Proxy. Ensure that any **input validation** performed on the **client-side is also performed on the server-side**.
{% endhint %}

* Data type validators available natively in web application frameworks \(such as [Django Validators](https://docs.djangoproject.com/en/1.11/ref/validators/), [Apache Commons Validators](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/package-summary.html#doc.Usage.validator), etc\).
* Validation against [JSON Schema](https://json-schema.org/) and [XML Schema \(XSD\)](https://www.w3schools.com/xml/schema_intro.asp) for input in these formats.
* Type conversion \(e.g. `Integer.parseInt()` in Java, `int()` in Python\) with strict exception handling.
* Minimum and maximum value range check for numerical parameters and dates, minimum and maximum length check for strings.
* The array of allowed values for small sets of string parameters \(e.g. days of the week\).
* Regular expressions for any other structured data covering the whole input string `(^...$)` and **not** using "any character" wildcard \(such as `.` or `\S`\). Developing regular expressions can be complicated, thus, it is recommended to follow a [comprehensive resource](https://www.regular-expressions.info/) when developing such a validation.

{% hint style="success" %}
As input validation is a widely used technique, ideally found in every other user interaction, you can [**learn more about it here**](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).
{% endhint %}

### 3.4. Escaping user-supplied input:

This technique should **only be used as a last resort** when none of the above are **feasible**. Input validation is probably a better choice as this methodology is frail **and we cannot guarantee** it will prevent all **SQLi** in all situations.

Imagine the following scenario: each DBMS supports one or more character escaping schemes specific to certain kinds of queries. If you then **escape all user-supplied input** using the **proper escaping scheme** for the database you are using, the **DBMS will not confuse that input** with **SQL code written by the developer**, thus avoiding **any** **possible SQL injection vulnerabilities**.

Usually, escaping the user input is on top of the already existing layer of defense, such as the **prepared statements** or **stored procedures**. In the following snippets, we make use of **language-specific commands** or specific database adapters that take care of escaping the query.

{% tabs %}
{% tab title="JavaScript" %}
```javascript
var SqlString = require('sqlstring');

var userId = 1;
var sql = SqlString.format('SELECT * FROM users WHERE id = ?', [userId]);
console.log(sql); // SELECT * FROM users WHERE id = 1
```
{% endtab %}

{% tab title="Python" %}
```python
from psycopg2 import sql
# here one cam make use of database adapters, such as psycopg2
def authenticate(request):
    email = request.POST['email']
    password = request.POST['password']
    with connection.cursor() as cursor:
        stmt = sql.SQL("""
            select 
                * 
            from 
                users 
            where
                (email = {email} and password = {password})
            """).format(
                email = sql.Identifier(email),
                password = sql.Literal(password),
        )
        cursor.execute(stmt)
        row = cursor.fetchone()
    
        if row:
            loggedIn = "Auth successful"
        else:
            loggedIn = "Auth failure"
        return HttpResponse("Logged In Status: " + loggedIn)

```
{% endtab %}

{% tab title="PHP" %}
```php
$stmt = $pdo->prepare('SELECT * FROM employees WHERE name = :name');
$stmt->execute(array('name' => $name));
foreach ($stmt as $row) {
    // do something with $row
    // PDO is the universal option; if you connect to databases other than MySQL
    // you can use driver-specific options.
}
```
{% endtab %}
{% endtabs %}

## 4. Takeaways:

**SQLI SHOULDNT EXIST!** Is what everyone says. And they are right! If every developer would:

1. **Prepare their SQL queries.**
2. **Validate & escape the user input.**
3. Write clean efficient code.\( =D \)

SQLIs will eventually go extinct! 

{% hint style="info" %}
You can find more details about this topic here:

* [OWASP Cheatsheet against SQLI](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
* [Preventing python SQLI](https://realpython.com/prevent-python-sql-injection/)
* [Query parametrization Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html)
* [Injection prevention Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
{% endhint %}

