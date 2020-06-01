---
layout: post
title: "W3af walkthrough Part 1"
date: 2013-06-13 08:49
comments: true
categories: [w3af, web-application-security, security]
---

w3af (Web Application audit and attack framework) is a framework for auditing and exploitation of web applications. In this series of articles we will be looking at almost all the features that w3af has to offer and discuss how to use them for Web application Penetration testing. In the first part of this series we will be working with w3af console and getting ourselves familiar with the commands. We will also be looking at the different types of plugins that w3af has to offer and discuss how to use them for optimal performance.

<!--more-->

Some of the major features of w3af are:

1.  It has plugins that communicate with each other. For eg. the discovery plugin in w3af looks for different url's to test for vulnerabilities and passes it on to the audit plugin which then uses these URL's to search for vulnerabilities.
2.  It removes some of the headaches involved in Manual web application testing through its Fuzzy and Manual request generator feature. It can also be configured to run as a MITM proxy. The requests intercepted can be sent to the request generator and then manual web application testing can be performed using variable parameters.
3.  It also has features to exploit the vulnerabilities that it finds.

It is important to understand that no automated web application scanner is perfect and false positives will always occur. With w3af the first and the foremost step is to make sure that we have the latest version. This is very important because w3af developers (Andres Riancho and the w3af team) are constantly fixing bugs and hence it is very important to make sure that we have the most bug free version. To open up w3af console, type in the command as shown in the figure below.

![1]({{site.baseurl}}/images/posts/w3af1/1.png)

w3af may ask you to update the version. It is advisable to keep updated with the latest version. Ok, so now that we are in the console, type in _help_ to look at the list of available commands.

![2]({{site.baseurl}}/images/posts/w3af1/2.png)

We can see the list of available options available to us. Type the _keys_ command to look at the various shortcuts keys available to us. I recommend you get familiar with them.

![3]({{site.baseurl}}/images/posts/w3af1/3.png)

Let's have a look at the plugins which are available in w3af. Type _plugins_. You can see the console output change to _w3af/plugins_. Type _back_ to go back or type _help_ to display the list of available plugins.

To know information about a specific plugins, just type _help pluginName_. For e.g if i want to know about the discovery plugin, i would type _help discovery_.

![5]({{site.baseurl}}/images/posts/w3af1/5.png)

We can see that there are about 9 types of different plugins.

**1)Discovery**- The discovery plugin helps in finding more Url's, forms etc to be used for vulnerability scanning. This information is then passed over to the audit plugin. There are a number of different discovery plugins like webSpider, spiderMan, hmap etc. All these plugins have a different function. A user can enable one or more plugins at the same time.

To see the discovery plugins, just type _discovery_.

![4]({{site.baseurl}}/images/posts/w3af1/4.png)

To find specific information about a particular plugin, just type _pluginType desc pluginname_. For e.g if i want to know more information about the spiderMan [index](index.html "index")plugin i would write the command _discovery desc spiderMan_.

![6]({{site.baseurl}}/images/posts/w3af1/6.png)

One of the important things to note here is that the spiderMan plugin has 2 configurable parameters. To set the configurable parameters, type in the following commands as shown in the figure below. As you can see from the figure below, i have set the listenPort to 55555.

![7]({{site.baseurl}}/images/posts/w3af1/7.png)

Here are some other commands that could be used.

1) discovery pluginType1, pluginType2 - Selects two plugins.

2) discovery all- Enables all the plugins (not advisable as it may take a long time to finish).

3) discovery !all - Removes all the enabled plugins.

4) list discovery enabled - Lists all the plugins currently enabled.

Here is a screenshot below showing some of these commands in action.

![8]({{site.baseurl}}/images/posts/w3af1/8.png)

Let's now run one of the discovery plugins. I will be using the hmap plugin in discovery to know the version of the server running on a remote host. As you can see from the figure below, i have enabled the hmap plugin.

![9]({{site.baseurl}}/images/posts/w3af1/9.png)

Once this is done, it is now time to give the location of the target server. Type _back_ to navigate back. Then type the following commands as shown in the figure below to set the target. As we can see, the target is set by the _set target target-address_ command.

![10]({{site.baseurl}}/images/posts/w3af1/10.png)

Once this is done, type _back_ to navigate back and the type _start_ to start the plugin. As we can see, w3af has figured out the version of Apache and php running on my server. We will discuss more features of the discovery plugin later.

![11]({{site.baseurl}}/images/posts/w3af1/11.png)

**2)Audit**-Audit plugins are used to detect vulnerabilities in the URL's or forms provided by the discovery plugins. This is where the interaction between plugins in w3af comes to use. The audit plugin has options for testing different types of vulnerabilities like xss, sqli, csrf etc. It does this by injecting different strings in its request and then looking for a specific value (corresponding to the input string) in the response. False positives may occur during this process. If i want to know how the sqli plugin works, i could type in the commands as shown in the figure below.

![12]({{site.baseurl}}/images/posts/w3af1/12.png)

Again, i can set the different configuration parameters while selecting a particular plugin. For e.g in the figure below i am increasing the number of checks while performing a XSS audit.

![13]({{site.baseurl}}/images/posts/w3af1/13.png)

**3)Grep** - The grep plugin is used to find interesting information in the requests and responses going through like email accounts, forms with file upload capabilities, hashes, credit card numbers, email addresses etc. You can set the type of information you want to look for by setting the appropriate plugin. Since the grep plugin only analyzes the request and response, it is important to have some kind of discovery plugin enabled for it to work. Otherwise grep plugins are of no use. As you can see in the figure below i have set grep to use the getMails plugin.

![14]({{site.baseurl}}/images/posts/w3af1/14.png)

**4)Brute force** - Brute force plugins can be used to brute force login forms as well as http-auth logins. Once the discovery plugin finds any form with form based input or an http-auth input it will automatically launch the brute force attack against it if the corresponding brute force plugin is enabled. Some of the important things to know about the brute force are the configuration parameters.

![15]({{site.baseurl}}/images/posts/w3af1/15.png)

It is advisable that you use your own configuration file for the list of usernames and passwords. Also be sure to take a look at some other options. As you can see in the figure below, i have set the option passEqUser to false simply because i don't think users wouldn't have their passwords as the same as their username.

![16]({{site.baseurl}}/images/posts/w3af1/16.png)

One of the other good configurable parameter is the useMails option. This options uses the email addresses that w3af finds (maybe through the grep plugin) to be one of the inputs for the username field. For e.g if one of the usernames is example@infosecinstitute.com, then the username tried would be example. This is another example of how the interaction between the different plugins could make the job much more effective.

**5)Output** - The output plugin helps us decide the format in which we want the output. w3af supports many formats like console, emailReport, html, xml, text etc. Again you can set various parameters here like the filename, verbosity etc. In the figure below, i have set _verbose_ to True as i want a very detailed report about the application that i am testing.

![17]({{site.baseurl}}/images/posts/w3af1/17.png)

**6)Mangle** - The mangle plugin is used to mangle with request and responses on the fly. It has only one plugin named sed (Stream editor) which is used to modify requests and responses using different regular expressions. The expressions should have a specific format. The usage is quite evident from the description.

![18]({{site.baseurl}}/images/posts/w3af1/18.png)

As you can see from the figure below, i have set the plugin to look for the string Yahoo and replace it with Google in the request header.

![19]({{site.baseurl}}/images/posts/w3af1/19.png)

**7)Evasion**- The evasion plugins uses various techniques to bypass WAF (Web application firewalls). For e.g one of the options rndHexEncode randomly encodes the url in hex format to avoid detection while the plugin fullWidthEncode does a full width encode of the Url to bypass Http content scanning systems using the vulnerability described [here](http://www.kb.cert.org/vuls/id/739224).

![20]({{site.baseurl}}/images/posts/w3af1/20.png)

**8)Auth** - Last but not the least, auth plugin is one of the most important plugins in w3af. It has only one type called generic. This is because while crawling on a target web application, if w3af hits a login form, then it needs to submit the credentials automatically in order to continue looking for information. By using this plugin, we can specify a predefined username/password that w3af should enter when it hits a login form. We need to specify all the parameters for generic in order for it to work successfully.

In the figure below i am setting options for w3af to successfully log in to DVWA (Damn vulnerable web application) which is located on the address http://10.0.1.24/dvwa

![21]({{site.baseurl}}/images/posts/w3af1/21.png)

## Conclusion

In this article we discussed about the plugins available in w3af and learnt how to work with the w3af console. In the upcoming articles in this series, we are going to discuss the following topics.

1.  Using different profiles
2.  Exploiting a vulnerability found by the audit plugin
3.  Using the Manual Request and Fuzzy request feature
4.  Using the Mitm proxy and the encoder/decoder features
5.  w3af scripting
6.  Optimizing w3af scans

Please drop a comment if you liked the article or if there is something about w3af that you want to see in the upcoming articles.

## References:

*   w3af User Guide  
    [http://w3af.sourceforge.net/documentation/user/w3afUsersGuide.pdf](http://w3af.sourceforge.net/documentation/user/w3afUsersGuide.pdf)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).