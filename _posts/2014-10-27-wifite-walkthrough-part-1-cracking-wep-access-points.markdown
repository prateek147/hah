---
layout: post
title: "Wifite Walkthrough part 1: Cracking WEP access points"
date: 2014-10-27 01:41
comments: true
categories: security
---

In this article series, we will look at a tool named Wifite suitable for automated auditing of wireless networks. Most of you who have experience in wireless pentesting would use tools like airmon-ng, aireplay-ng, airodump-ng, aircrack-ng to crack wireless networks. This would involve a sequence of steps, like capturing a specific numbers of IV's in case of WEP, capturing the WPA handshake in case of WPA etc, and then subsequently using aircrack-ng to crack the password required for authentication to the network. Wifite aims to ease this process by using a wrapper over all these tools and thus making it super easy to crack Wifi networks.

<!-- more -->

Here is a list of features of Wifite as per its official [homepage](https://code.google.com/p/wifite/).

*   sorts targets by signal strength (in dB); cracks closest access points first
*   automatically de-authenticates clients of hidden networks to reveal SSIDs
*   numerous filters to specify exactly what to attack (wep/wpa/both, above certain signal strengths, channels, etc)
*   customizable settings (timeouts, packets/sec, etc)
*   "anonymous" feature; changes MAC to a random address before attacking, then changes back when attacks are complete
*   all captured WPA handshakes are backed up to wifite.py's current directory
*   smart WPA de-authentication; cycles between all clients and broadcast deauths
*   stop any attack with Ctrl+C, with options to continue, move onto next target, skip to cracking, or exit
*   displays session summary at exit; shows any cracked keys
*   all passwords saved to cracked.txt
*   built-in updater: ./wifite.py -upgrade

Before we start using wifite, make sure you have a proper wireless card that supports packet injection. If you don't have one, i would suggest that you buy [this](http://www.amazon.com/Alfa-AWUS036H-802-11b-Wireless-network/dp/B002WCEWU8) card.

Note that there is a bug in Wifite that may or may not be there in your particular version of Wifite. The bug basically doesn't aireplay-ng to function properly and displays an error like _aireplay-ng exited unexpectedly_ . In order to fix this, you will have to make slight modifications in the code of wifite. You can install gedit (apt-get install gedit) which is a text editor and then edit the wifite python script (found in /usr/bin/wifite) using the steps mentioned [here](https://code.google.com/p/wifite/issues/detail?id=127). To open wifite, use the command _gedit /usr/bin/wifite_. This will open up the source code of wifite. Then replace every occurence of _cmd = ['aireplay-ng',_ with _cmd = ['aireplay-ng','--ignore-negative-one',_

Wifite can be found under _Applications -> Kali Linux -> Wireless Attacks -> 802.11 Wireless Tools_. Also, note that if you are running wifite in a different VM than Kali Linux, then you have to make sure that tools like airmon-ng, aireplay-ng, airodump-ng, aircrack-ng are already installed on that system. This is because Wifite is nothing but a wrapper over all these tools. Before we even start using Wifite, it is better to update to the latest version.

![1]({{site.baseurl}}/images/posts/wifite1/1.png)

In my case, i already have the latest version. In this tutorial, we will be targeting a simple Wifi network with WEP encryption. Just using the command _wifite -h_ will give you a list of all the commands.

![2]({{site.baseurl}}/images/posts/wifite1/2.png)

A very tempting option would be _-all_ which tries to attack every network that it finds. We will try it in later articles in this series. However, first lets take a look at all the targets that we have. To do that, use the command _wifite -showb_

![3]({{site.baseurl}}/images/posts/wifite1/3.png)

Once this is done, we can see that wifite has put our network interface card into monitor mode (using airmon-ng) and started to look for clients. After a few more seconds, it will start displaying the list of access points.

![4]({{site.baseurl}}/images/posts/wifite1/4.png)

Note that as it is mentioned in its feature list (automatically de-authenticates clients of hidden networks to reveal SSIDs), this list will also include hidden access points. Hence, wifite can also be used to find hidden access points. In this case we will attack an access point with the BSSID 00:26:75:02:EF:65 that i have set up for testing purposes. The access point has a simple WEP password _1234567890_.

![5]({{site.baseurl}}/images/posts/wifite1/5.png)

To start attacking an access point, just press _Ctrl+C_. Wifite will now ask you to choose a target number from the list. The target number for my test network is 1, so let me enter that. Note that if you press _Ctrl+C_ again, it will quit Wifite.

![6]({{site.baseurl}}/images/posts/wifite1/6.png)

You can now see that Wifite will start attempting to crack the WEP access point using the different known techniques for cracking WEP encryption. After some unsuccessful tries, it has finally begun to start attacking the access points using different techniques for cracking WEP.

![11]({{site.baseurl}}/images/posts/wifite1/11.png)

Once enough IV's are being captured, it will automatically start cracking the password.

![12]({{site.baseurl}}/images/posts/wifite1/12.png)

As we can see, Wifite has successfully figured out the WEP key for the access point. Wifite is an extremely useful tool for cracking wireless networks. As i mentioned previously, you need to have all the tools like airmon-ng, aireplay-ng, airodump-ng, aircrack-ng already installed on your system. To further prove the point, let's dive into the source code of Wifite.

![10]({{site.baseurl}}/images/posts/wifite1/10.png)

As we can see, the python code has mentions of calling aireplay-ng. Hence, it is recommended to run Wifite inside Kali linux.

In the next article, we will look at some advanced usage options of Wifite.