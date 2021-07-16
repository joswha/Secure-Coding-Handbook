# Directory Traversal

## 1. Introduction:

Directory traversal \(also known as file path traversal\) is a web security vulnerability that allows an attacker to read arbitrary files on the server that is running an application. This might include application code and data, credentials for back-end systems, and sensitive operating system files. In some cases, an attacker might be able to write to arbitrary files on the server, allowing them to modify application data or behavior, and ultimately take full control of the server.

More on Directory Traversal, [from an attacker's perspective](https://portswigger.net/web-security/file-path-traversal)[.](https://portswigger.net/web-security/file-path-traversal)

## 2. Typical vulnerable code:

There are only so many cases where **directory traversal** is possible, and most of the times, the code behind the **vulnerable application** looks like the following:

```java
private static final String BASE_PATH = "/storage/items/images";

private void getProfileImage(HttpServletRequest request, 
HttpServletResponse response) throws IOException {
    
    String folderName = request.getParameter("folder");
    String fileName = request.getParameter("file");
    String path = BASE_PATH + folderName + fileName;

    File file = new File(path);
    
    buildResponse(response, file);
}

private void constructResponse(HttpServletResponse response, 
File file) throws IOException {
    
    response.setContentType("image/png");
    
    OutputStream os = response.getOutputStream();
    
    // Notice there is no extra validation on the path of the file, it is read 
    // straight away.
    BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
   
    byte[] buffer =  new byte[1024];
    int read;

    while ((read = bis.read(buffer)) != -1) {
        os.write(buffer, 0, read);
    }

    bis.close();
    os.flush();
    os.close();
}
```

We read off the fact that the path is constructed\(based on the `BASE_PATH` and the request-received **file** and **folder name\).**

**Guessing this,** an attacker can input a malicious path into the filename, for example `../../../etc/passwd` which would translate into `/storage/items/images/../../../etc/passwd`\(assuming we're dealing with a UNIX system\) which then is interpreted by the system as `/etc/passwd` thus leaking the `passwd` file.

## 3. Mitigations:

### 3.1. Validating absolute path:

The most effective way to prevent **Directory Traversal** vulnerabilities is to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior more securely.

However, if this is not possible, the application should perform strict **input validation** against **parameters** that are intended to be used for file system operations. These include path validation and absolute path checking of user-supplied data.

Simply put, we can check the absolute path of the requested file, and should that not match with our hardcoded base path, ignore it altogether.

```java
private static final String BASE_PATH = "/storage/items/images";

private void getProfileImage(HttpServletRequest request, 
 HttpServletResponse response) throws IOException {
    
    String folderName = request.getParameter("folder");
    String fileName = request.getParameter("file");
    String path = BASE_PATH + folderName + fileName;

    File file = new File(path);
    
    String canonicalPath = file.getCanonicalPath();

    // Check whether the given path corresponds to the base path
    //(where the image files are stored)
    if(canonicalPath.startsWith(BASE_PATH)) {
        buildResponse(response, file);
    } else {
        throw new GenericException("Access denied.");
    }
}

private void constructResponse(HttpServletResponse response,
 File file) throws IOException {
    
    response.setContentType("image/png");
    
    OutputStream os = response.getOutputStream();
    BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
   
    byte[] buffer =  new byte[1024];
    int read;

    while ((read = bis.read(buffer)) != -1) {
        os.write(buffer, 0, read);
    }

    bis.close();
    os.flush();
    os.close();
}
```

### 3.2. Using a File-Hosting Service:

We have discussed a similar approach to actually **storing** the files in a secure way on a CDN/ cloud service here:

{% page-ref page="../auxiliary/file-upload.md" %}

As a follow up, retrieving these files would by default be done in a secure and efficient way. They are really easy to implement and play around with, so you should definitely use them from now on.

### 3.3. Indirect file references:

Should storing files on the **local server really be necessary,** the most secure way of **defusing directory traversal attacks is via indirection**: you assign each file an arbitrary ID that corresponds to a **filepath**, and then have **all URLs reference each file by that ID**. This can be done, for example, **using a database**\(where you keep the reference between the **path of the file** and its **relative id**\)

## 4. Conclusions:

**Validating the absolute path** represents the basis of defending against directory traversal. Your go-to defense however is **using a CDN or any other filestoring online/ cloud systems**. Should storing files on the local server be one of your requirements, remember that **indirect file referencing** is a very powerful technique and you should leverage it.

{% hint style="info" %}
You can find more details about this topic here:

* [What is directory traversal?](https://portswigger.net/web-security/file-path-traversal)
* [Directory Traversal exercise.](https://application.security/free-application-security-training/owasp-top-10-directory-traversal)
{% endhint %}

