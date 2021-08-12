# Vulnerable Dependency Management

## 1. Introduction:

**Development teams rarely perform code reviews on third-party dependencies,** but the libraries and tool kits we use are often a source of software vulnerabilities. As a developer, you need to ensure code written by **other** people is not making **your system** insecure.

## 2. Protection:

Being mindful when using dependencies on your projects and software in general is key to keeping your system secure. With this in mind, when working with "third-party code" every developer should follow the following points:

* **Automate the build and deployment processes.** You need to make sure you actually **KNOW what code is running and when;** This means declaring all third-party libraries within build scripts or dependency management systems/ building and deploying from source control and even keeping records of deployment logs.
* **Never trust private dependencies.** Be careful how you configure the precedence of repositories in your build process, since [dependency confusion](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610) attacks - where an attacker uploads a **malicious** copy of a **private dependency to a public repository** - have caused a **lot of trouble worldwide**.
* **Deploy known-good versions of software.** Dependency management tools often allow you to leave the version of each dependency indeterminate, which is shorthand for **“grab the latest available version at build time.”** Try to avoid this - upgrade versions deliberately, when you have had chance to review the release notes, and pin the dependency versions in your code.
* **Use dedicated tools to scan your dependency tree for security risks.** Most programming languages and utilities are able to spot compromised dependencies. Consider using one or more of the following:
  * [Github security alerts](https://help.github.com/en/github/managing-security-vulnerabilities/about-alerts-for-vulnerable-dependencies).
  * [GitLab security scanning](https://docs.gitlab.com/ee/development/integrations/secure.html).
  * [`npm audit`](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities) and [`retire.js`](https://retirejs.github.io/retire.js/) for Node.
  * [`bundler audit`](https://github.com/rubysec/bundler-audit) for Ruby.
* **Keep on top of security bulletins.** Make sure your team is on the lookout for security announcements for the software you use. This can mean signing up for mailing lists, joining forums, or following library developers on social media. The development community is often the first become aware of security issues.
* **Make penetration testing and code reviews part of your development lifecycle.** Penetration testing tools will attempt to take advantage of known exploits, checking whether your technology stack contains vulnerable components. Code reviews, on the other hand, allow for a constant reminder of what the specific code is supposed to to and how it does that\(eg. what dependencies it uses and so on\).
* **Using dependency management tools.** Since they simplify a lot of the developer's job, you should consider using one of them. Most dominant programming languages have their own dependency management tools, such as:
  * [Bundler](http://bundler.io/) for Ruby Gems.
  * [Pip](https://packaging.python.org/installing/#use-pip-for-installing) for Python Packages.
  * [NPM](https://docs.npmjs.com/) for Node Modules.
  * [Maven](https://maven.apache.org/what-is-maven.html) and [Gradle](https://docs.gradle.org/current/userguide/tutorial_java_projects.html) for Java jars.
  * [NuGet](https://www.nuget.org/) for .NET.
  * [Composer](https://getcomposer.org/) for PHP.

## 3. Takeaways:

To sum up, **component-based vulnerabilities** occur when a web application component is unsupported, out of date, or vulnerable to a known exploit.

To effectively mitigate against ﻿**component-based vulnerabilities**, developers must regularly audit software components and their dependencies, making sure the third-party libraries and software dependencies are always up-to-date.

Product teams must further establish security policies governing the use of third-party libraries, such as passing security tests, and regular patching and updating of application dependencies.

{% hint style="info" %}
You can find more details about this topic here:

* [Components with known vulnerabilities.](https://application.security/free-application-security-training/owasp-top-10-components-with-known-vulnerabilities)
* [Vulnerable Dependency Management Cheat](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html).
* [Securing Your Dependencies.](https://www.hacksplaining.com/prevention/toxic-dependencies)
* [Dependency Confusion.](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)
{% endhint %}



