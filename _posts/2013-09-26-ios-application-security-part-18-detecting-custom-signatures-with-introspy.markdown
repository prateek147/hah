---
layout: post
title: "iOS Application Security Part 18 – Detecting custom signatures with Introspy"
date: 2013-09-26 17:18
comments: true
categories: security
---

In the previous article, we looked at how we can use Introspy for Black-box assessment of iOS applications. In this article, we will look at how we can use Introspy to set up our own custom signatures and detect them in an application trace. Setting up our own predefined signatures could be useful for cases where you have a found a method in a particular application that seems of particular interest to you and you want to know when it is being called. Introspy already has a list of predefined signatures that it uses to flag vulnerabilities or insecure configurations. However, it also allows us to add our own signatures.

<!-- more -->

You can find the predefined signatures in Introspy in the signatures.py file inside the analyzer folder.

![1]({{site.baseurl}}/images/posts/ios18/1.png)

From here, we can see that a signatures consists of a title, description, a severity level and a filter which consists of the method calls that correspond to the signature. So let's look at a sample signature.

![2]({{site.baseurl}}/images/posts/ios18/2.png)

Over here, you can see a signature that checks whether the application uses Pasteboards or not. Pasteboards are generally very insecure as they can allow an application to copy some data from the Pasteboard into their application. Hence this signature makes sense. You can see that the filter consists of two values, _classes_to_match_ and _methods_to_match_. You can also specify a parameter _args_to_match_ in your signature. So from this signature, it is pretty much clear that these following method implementations will match the above signature.

*   UIPasteboard *pasteboard = [UIPasteboard generalPasteboard];
*   UIPasteboard *pasteboard = [UIPasteboard pasteboardWithName:@"XYZ" create:YES];
*   UIPasteboard *pasteboard = [UIPasteboard pasteboardWithUniqueName];

Another signature shown in the image below checks for methods that bypass credential validation while connecting to a remote server. This happens in cases where you are using a self singed SSL certificate and would like to trust it everytime without any kind of validation.

![3]({{site.baseurl}}/images/posts/ios18/3.png)

For any LibC signature, just set the _classes_to_match_ attribute as _C_.

![4]({{site.baseurl}}/images/posts/ios18/4.png)

Now lets understand a signature that has some arguments also with it as a filter. The filter can be defined using 3 classes which can be found in the file Filters.py. These classes are _ArgumentsFilter_, _ArgumentsNotSetFilter_ or _ArgumentsWithMaskFilter_. Here are some screenshots from the code itself that define the purpose of these classes.

![8]({{site.baseurl}}/images/posts/ios18/8.png) ![9]({{site.baseurl}}/images/posts/ios18/9.png) ![10]({{site.baseurl}}/images/posts/ios18/10.png)

Here is a signature written in the Signatures.py file that detects the scenario when some data is written to the keychain without a secure protection domain (pdmn). As you can see, both the _ArgumentsFilter_ and _ArgumentsNotSetFilter_ filters have been used to detect signaturres. The _ArgumentsFilter_ signature is used to find pdmn's that are insecure whereas _ArgumentsNotSetFilter_ is used to find cases where no accessibility option is provided and hence defaults to kSecAttrAccessibleAlways.

![11]({{site.baseurl}}/images/posts/ios18/11.png)

Now lets add a custom signature to the signature.py file. In this case, we are checking for the case whenever someone gets a string saved in NSUserDefaults.

![5]({{site.baseurl}}/images/posts/ios18/5.png)

Now run the python script introspy.py file on the saved database file.

![6]({{site.baseurl}}/images/posts/ios18/6.png)

In the report under Potential Findings, you will see that our signature was identified in many different places.

![7]({{site.baseurl}}/images/posts/ios18/7.png) **Conclusion**

In this article, we looked at how we can use Introspy to set up our own custom signatures and detect them in an application. Using these custom signatures when performing static analysis of these applications can be very useful if you are looking to track some specific method implementations.

**References**

*   Introspy  
    [https://github.com/iSECPartners/introspy](https://github.com/iSECPartners/introspy)