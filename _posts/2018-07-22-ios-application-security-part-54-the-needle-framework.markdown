---
layout: post
title: "iOS Application Security Part 54 - The Needle Framework"
date: 2018-07-30 11:47
comments: true
categories: security 
---
In this article, we will talk about another framework for assessing iOS apps named [Needle.](https://github.com/mwrlabs/needle/) Released by MWR labs and written by Marco Lancini, it provides a lot of modules that with help automate a lot of the tasks while doing iOS security assessments. Needle requires a jailbroken device and uses an agent installed on the jailbroken device that communicates with the host installed on the computer. At the time of writing of this article, Needle supports devices until iOS 10 only.
<!--more-->

To install Needle, add the source http://mobiletools.mwrinfosecurity.com/cydia/ in Cydia and then search for NeedleAgent and install it.

![1]({{site.baseurl}}/images/posts/ios54/1.PNG) 

Open the NeedleAgent app and make sure it is listening.

![2]({{site.baseurl}}/images/posts/ios54/2.PNG)

On your computer, clone the latest version of Needle and install all the dependencies . A detailed installation guide can be found [here](https://github.com/mwrlabs/needle/wiki/Installation-Guide).

![3c]({{site.baseurl}}/images/posts/ios54/3c.png)

Make sure the needle agent is running in foreground on the device. Run the _show options_ command to see all the list of global options.

![4c]({{site.baseurl}}/images/posts/ios54/4c.png)

Make sure to set the correct PASSWORD option to let Needle connect to the device. Once you have configured these settings, run the _shell_ command to get a shell on your device. Another important global option that you can set is the OUTPUT_FOLDER option. You can then use the _exit_ command to exit out of the shell and back into the needle interpreter.

![5c]({{site.baseurl}}/images/posts/ios54/5c.png)

Running the command _show modules_ will list all the modules that Needle supports.

![6c]({{site.baseurl}}/images/posts/ios54/6c.png)

You can use any module with the _use modulename_ command, and the _run_ command will execute the module for you. If you want to analyze any specific app, you can see the app bundle id as a global parameter. If this is not set, needle will display a prompt to let you choose whichever app id you want.

![7c]({{site.baseurl}}/images/posts/ios54/7c.png) ![8c]({{site.baseurl}}/images/posts/ios54/8c.png)

The following module will tell you the URL schemes the app registers to by copying the info.plist file.

![9c]({{site.baseurl}}/images/posts/ios54/9c.png)

And this one will give the MDM user settings for the device.

![10c]({{site.baseurl}}/images/posts/ios54/10c.png)

Running the _info_ command after selecting a module will give you the details about that particular module.

![11c]({{site.baseurl}}/images/posts/ios54/11c.png)

Needle also has many different modules for working with Frida.

![12c]({{site.baseurl}}/images/posts/ios54/12c.png)

And in some cases the modules might also have certain options that need to be configured.

![13c]({{site.baseurl}}/images/posts/ios54/13c.png)

Here is the full list of modules. It is important to note that not all of these features work for iOS 10, and thus at the time of writing this article, needle doesn't support iOS 10 completely.

<pre>	[needle] > show modules

	  _Templates
	  ----------
	    _templates/template_background
	    _templates/template_base
	    _templates/template_frida
	    _templates/template_frida_script
	    _templates/template_static

	  Binary
	  ------
	    binary/info/checksums
	    binary/info/compilation_checks
	    binary/info/metadata
	    binary/info/provisioning_profile
	    binary/info/universal_links
	    binary/installation/install
	    binary/installation/pull_ipa
	    binary/reversing/class_dump
	    binary/reversing/class_dump_frida_enum-all-methods
	    binary/reversing/class_dump_frida_enum-classes
	    binary/reversing/class_dump_frida_find-class-enum-methods
	    binary/reversing/shared_libraries
	    binary/reversing/strings

	  Comms
	  -----
	    comms/certs/delete_ca
	    comms/certs/export_ca
	    comms/certs/import_ca
	    comms/certs/install_ca_burp
	    comms/certs/install_ca_mitm
	    comms/certs/list_ca
	    comms/certs/view_cert
	    comms/proxy/pinning_bypass_frida
	    comms/proxy/proxy_regular

	  Device
	  ------
	    device/agent_client
	    device/clean_storage
	    device/dependency_installer
	    device/hosts
	    device/list_apps

	  Dynamic
	  -------
	    dynamic/detection/jailbreak_detection
	    dynamic/detection/script_jailbreak-detection-bypass
	    dynamic/ipc/open_uri
	    dynamic/memory/heap_dump
	    dynamic/monitor/files
	    dynamic/monitor/pasteboard
	    dynamic/monitor/syslog
	    dynamic/watch/syslog

	  Hooking
	  -------
	    hooking/cycript/cycript_shell
	    hooking/cycript/cycript_touchid
	    hooking/frida/frida_launcher
	    hooking/frida/frida_shell
	    hooking/frida/frida_trace
	    hooking/frida/script_anti-hooking-check
	    hooking/frida/script_dump-ui
	    hooking/frida/script_hook-all-methods-of-class
	    hooking/frida/script_hook-method-of-class
	    hooking/frida/script_touch-id-bypass
	    hooking/theos/list_tweaks
	    hooking/theos/theos_tweak

	  Mdm
	  ---
	    mdm/effective_user_settings

	  Static
	  ------
	    static/code_checks

	  Storage
	  -------
	    storage/backup/icloud_content_frida
	    storage/caching/keyboard_autocomplete
	    storage/caching/screenshot
	    storage/data/container
	    storage/data/files_binarycookies
	    storage/data/files_cachedb
	    storage/data/files_plist
	    storage/data/files_sql
	    storage/data/keychain_dump
	    storage/data/keychain_dump_frida

	[needle] >
</pre>

**References**

1.  https://github.com/mwrlabs/needle/