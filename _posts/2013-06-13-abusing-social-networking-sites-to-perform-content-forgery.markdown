---
layout: post
title: "Abusing Social Networking Sites to Perform Content Forgery"
date: 2013-06-13 22:38
comments: true
categories: [web-application-security]
---

Web Application vulnerabilities in social networking sites is very common these days. In this article we will be discussing a vulnerability found in Social networking sites because of which it is possible to spoof the content shown to the user. Basically whenever someone wants to share, post or send a link on Facebook or some other social networking site, a request goes through from their servers to the link which the user wants to share. This happens because Facebook (or that particular social networking site) wants to display a quick snapshot of what appears in the link to the user. However, these requests by social networking sites are easily identifiable because of the user-agent field in the headers of the incoming requests to the server or through their source IP address that resolves to a particular domain name. Hence it is possible for a malicious person to differentiate between the requests coming from the social networking sites and those coming from the users. The attacker can then display a simple image when the request is coming from Facebook so that on Facebook the snapshot appears to be that of a simple image. However when the user clicks on the link on Facebook, the attacker can know that the request is from the user by checking the user-agent field and redirect him to a malicious website.

<!--more-->

## That Easily Identifiable Request

Let's say i want to share a link http://google.com on my Timeline. Whenever i type the link, Facebook automatically identifies it as a url and sends a request to http://google.com. This is because it wants to display a quick snapshot of what actually appears to be in the link. As we can see from the image below, Facebook has displayed a quick snapshot of how http://google.com appears like today.

![1]({{site.baseurl}}/images/posts/content-forgery/1.png)

These requests to the google.com server are easily identifiable because the user-agent of the incoming request is an indication of the social networking site being used or the IP address of the incoming request resolves to a particular domain name. For e.g Facebook uses a custom user-agent with the name "facebookexternalhit" and have their IP addresses resolve to tfbnw.net whereas Google+ uses a custom user-agent with the name "Feedfetcher-Google".

By using some custom php code like "$_SERVER['HTTP_USER_AGENT']" to get the user-agent value and "gethostbyaddr($_SERVER['REMOTE_ADDR'])" to resolve the IP address we can easily identify whether the request is coming from a social networking site like Google+ or Facebook or just a regular user.

## Implementation

One of the most important things in order to implement this is to have a domain and an access to a publicly reachable server. We will have a .jpg on our server which will have the php code to identify the user-agent or IP address and display the response accordingly. However in order for the php code to be run in the jpg file, the jpg file must be interpreted as a php file by the server. The following two lines when added in the .htaccess file will serve our purpose. The code has been taken from http://www.blackhatacademy.org/security101/Facebook

``` php AddType x-httpd-php .jpg AddHandler application/x-httpd-php .jpg ```

The first line assures that any .jpg file will be interpreted as a php file whereas the second line assures that any file with the externsion .jpg will be treated as a php program.

Once this is done, we need to have the php code in our .jpg file on the server. The following lines of code demonstrate the exploit code for Facebook, Google+ and Websense. The code has been taken from [http://www.blackhatacademy.org/security101/Facebook](http://www.blackhatacademy.org/security101/Facebook) with the comments and the code a bit modified.

``` php ```

The code is pretty simple to understand. We check the user-agent, and the IP address of the incoming request. If the user-agent is identified as that of either Facebook, Google Plus or Websense then we know that this could be a request to take a snapshot of the url. Hence we display an image present on our server. Note that we also need to modify the Content-Type of the response to make it appear as an image. Hence on the social networking site, say Facebook, the snapshot of the url appears to be an image. However when the user clicks on the url, this time the user-agent is not of Facebook, Google+ or Websense rather it is the user-agent of the user itself. The php code identifies this as a regular user and redirects him to his malicious website (in this case evilsite.com).

Similarly the POC code for Reddit is demonstrated below. This code has been taken from [http://www.chokepoint.net/?id=5](http://www.chokepoint.net/?id=5)

``` php ```

Their is also a POC facebook application available at [http://www.chokepoint.net/?id=5](http://www.chokepoint.net/?id=5). It allows us to input an image and a redirect URL. As you can see below, i have given the input image as the image of the Google Logo whereas the redirect url is http://youtube.com.

![2]({{site.baseurl}}/images/posts/content-forgery/2.png)

Once this is done, click on Submit. You will see a like button, a send button as well as a Reddit this! button appear. You can now send this request to yourself to check this vulnerability.

![4]({{site.baseurl}}/images/posts/content-forgery/4.png)

Here is how the sent message looks like. As you can see, the snapshot contains the image of the google URL.

![3]({{site.baseurl}}/images/posts/content-forgery/3.png)

However, when we click on the url, we will be redirected to youtube.com. Try this yourself to see !

## Protection Against Content Forgery

One of the ways to prevent from Content Forgery is to spoof the user-agent field to look like it is coming from a normal user. Another method could be to spoof the user-agent field as the user-agent of the user itself (i.e the user who is sharing or posting the link).

## Conclusion

In this article we looked at a vulnerability which is caused due to the fact that the request made by Social Networking sites are easilty identifiable. These requests can hence be filtered out from the actual requests by the users. We looked at how this allows us to spoof the content which is actually appearing on the site. This could be used to redirect the users to malicious web pages through these Social networking sites simply because the original snapshot of the url looks pretty simple and unharming.

## References

1.  Facebook-Security 101  
    [http://www.blackhatacademy.org/security101/Facebook](http://www.blackhatacademy.org/security101/Facebook)

2.  Facebook and Reddit Content Forgery  
    [http://www.chokepoint.net/?id=5](http://www.chokepoint.net/?id=5)

3.  XSCF  
    [http://www.blackhatacademy.org/security101/XSCF](http://www.blackhatacademy.org/security101/XSCF)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).