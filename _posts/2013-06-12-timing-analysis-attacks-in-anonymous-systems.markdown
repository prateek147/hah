---
layout: post
title: "Timing Analysis Attacks in Anonymous Systems"
date: 2013-06-12 05:32
comments: true
categories: [security, timing-analysis-attacks]
---
Anonymous systems are used to allow users to surf the web, communicate with servers anonymously. Some of the popular Anonymity service providers are TOR, GTunnel etc. The basic idea is to hide the identity of the user. However it is important to ensure that the efficiency of the anonymous system in not decreased in the process which could depend on numerous factors like latency, degree of anonymity etc. The communication between the sender and the receiver happens through a set of routers, often referred to as a mix or node, whose job is to hide the relation between the incoming and the outgoing packets through it by using various techniques like using encryption, adding delays, adding cover traffic etc.<!--more--> In Timing analysis attacks we assume that the attacker has access to a particular set of mixes, i.e the attacker is a part of the network. By studying the timing of the messages going through the mixes, it is possible for him to determine the mixes that form a communication path. If the first and last mixes in the network are owned by the attacker, then it is possible for him to figure out the identities of the sender and the receiver. Some of the anonymous systems use a technique known as constant rate cover traffic, in which some extra cover traffic is sent along with the normal traffic so that it appears that the messages are sent at a constant rate through each path, thereby making it difficult for the attacker to find any correlation between the messages. In the past few years, many attacks and defenses have been proposed against Timing analysis attacks, some of which we will be discussing in this article.

## Anonymous systems

**a) TOR** -Tor is a software that provides online anonymity to the users by not revealing their identity. The communication between the sender (client) and the receiver (destination) happens through a bunch of Tor nodes that form together a network. These nodes are actually the systems of the various users around the world running TOR who opt to be a part of the network. When a user wants to visit a particular website, TOR chooses a random path which goes through these nodes and finally to the destination. Note that the traffic between these nodes is encrypted and follows the Onion routing protocol. In Onion routing, a message is encrypted a number of times and then sent to the network containing the nodes. In each node the message is decrypted once through which it receives further routing instructions and hence moves over to the next node. This process is similar to peeling an onion layer by layer, and hence the name Onion Routing. This ensures that each node does not know about the sender and the receiver. Also it becomes difficult for an attacker performing traffic analysis on the network to figure out the identity of the sender and the receiver.

Tor can be downloaded from [here](https://www.torproject.org/download/download.html.en). The following figure shows TOR running on my system using _Vidalia_. Vidalia is a kind of GUI for TOR. It lets us perform various operations like Start/Stop Tor, View the Network of users which form a part of the TOR network, Setup relaying and so on.

![Img1]({{site.baseurl}}/images/posts/timing/img1.png)

We can also set up our system to be a part of the TOR network.You can find more information about that [here](https://www.torproject.org/docs/tor-doc-relay.html.en).Click on _View the Network_ to see the number of systems acting as relays for the TOR network.

![Img2]({{site.baseurl}}/images/posts/timing/img2.png)

Please note that it is possible to run your system as a middle relay or an exit relay. The exit relay is the last relay through which the traffic on a certain communication path passes through before reaching the final destination. There can be many exit nodes on the whole network. In case an illegal activity is performed by someone using TOR, the exit relay could be blamed for that because the IP address of the exit relay will appear to be the source address in the logs of the victim. Hence people who want to run exit relays should be aware of these kind of things.

**b) GTunnel** - GTunnel is another anonymity system for Windows released by Garden Networks in 2007\. It works as a local HTTP proxy and SOCKS proxy. When we use this proxy for our browser, the traffic is directed through the GTunnel servers before reaching the original destination in the standard mode. There are other modes too in which we can use GTunnel, for example the _Skype Mode_, in which GTunnel first connects through the Peer to Peer network of Skype and then to the GTunnel servers, as well as the _Tor mode_, in which Gtunnel connects through Tor nodes to the GTunnel servers and then to the final destination. Note that the traffic is encrypted throughout the communication path.

## Timing Analysis attacks

**a) Watermarking** - In this technique, the attacker actively injects the message in a flow with a specific pattern. If the pattern is then observed in some other flow in the output stream of a mix, then it can be assumed that the two flows have a common path. This technique has proven to be quite successful against anonymous systems. One of the other advantages of Watermarking is that it is not very computationally expensive to perform.

**b) Flow Correlation attack** - If successfully perfomed, this attack can allow the attacker to identify the path of the flow and even reveal the identity of the sender and the receiver. The idea is to analyze one particular mix, see all the traffic that it is receiving at a particular port, and then find which output ports on the same mix is carrying the traffic corresponding to the traffic at the input port. This can be achieved by noting the similarity between the traffic at the input and the output port, which could be based on the timing of the packets, packet size etc.

**c) Selective Cross Correlation attack** - This attack is a bit more effective than the Correlation attack. Some of the mixes add some dummy traffic (also called cover traffic) to prevent from timing analysis attacks. In Selective Cross Correlation attack, we assume the incoming and outgoing traffic as two streams, and divide both these streams into non-overlapping windows. We then try to compare the packets in the incoming window in the input stream with the corresponding packets in the outgoing window in the output stream. In case the traffic in the outgoing window is more than the incoming window, then we can clearly say that some dummy traffic has been added in this case. We then remove all the windows in which we think cover traffic has been used from our test case and perform cross correlation on the other windows only thereby giving us a better result.

## Defenses Against Timing Analysis Attacks

To counter the threat of Timing analysis attacks , a number of defenses and algorithms have been proposed in the last few years. The solution is to choose a tradeoff between the best defense and the various factors which affect the efficiency of an anonymity system such as Latency, Bandwidth and anonymity. Also the topology of the network is an important consideration in this case. In this section we will go through the various defenses that have been proposed against timing analysis attacks.

**a) Adaptive Padding** - In case of adaptive padding, arbitrary packets are inserted into the stream by the mixes in between in order to destroy the results for timing analysis attacks. This prevents the attacker from linking the incoming and outgoing streams to a certain extent. Padding ratio is a term used to signify the no of dummy packets used per original packet. For e.g if the padding ratio is 0.5, then it means that there is 1 dummy packet for every 2 real packets. Adaptive padding works pretty well against passive timing analysis attacks. It can also work well with active timing analysis attacks, in which specific patterns are added in streams in order to fingerprint them for later use, but with a little compromise of a certain increase in the latency.

**b) Defensive Dropping** - In Defensive Dropping the sender sends the message with some dummy packets and the intermediate mixes are instructed to drop those packets. If this process is repeated several times for different streams, then it reduces the correlation between the incoming and outgoing streams and hence the probability of an attacker figuring out the path of the flow. These packets are randomly selected to be dropped. Note that no one mix is instructed to drop the packets, rather a number of intermediate mixes are instructed to drop the packets.

**c) Gamma Buffering** - This technique involves buffering some packets before passing them over from one mix to another. For e.g if the number of incoming connections to a node is p and the value of Gamma is γ, then according to the algorithm, the node must buffer a total of γ*p no of packets before passing them through. However some of the disadvantages of Gamma Buffering is that it could lead to additional delays. For e.g the required number of packets (γ*p) may arrive earlier in a stream than the other. Hence there would be an extra delay in the stream receiving lesser number of packets which could increase further as the packets flow through other nodes. This problem will not occur on high traffic networks though. These variable delays will destroy many of the timing characteristics and watermarks that an attacker must have injected and will be looking for in the outgoing streams.

## Testing

These kind of attacks can be tested by setting up a private TOR network. It is always a challenging task to test these attacks mainly because they involve us to control a lot of systems at once. [Planetlab](http://www.planet-lab.org/) and [Deterlab](http://www.isi.deterlab.net/index.php3) make this very easy for us. Planetlab consists of a group of computers contributed from different sites across the world to serve as a testbed for performing experiments that involve a large number of nodes. Different universities and corporations across the world provide their own systems to be used for performing experiments, development and research. Access is not provided to all users but only to those members who are a part of an institution that is a member of the PlanetLab Consortium. Every institution which is a part of Planetlab has a Principal Investigator that should approve accounts for the members of their own institution. Deterlab is another testbed for conducting network security related experiments. To have access to Deterlab the head of the project must register for a new project on Deterlab's website. Different projects could be run on the same nodes without affecting each other. We can also set up the topology of the {{site.baseurl}}/images/posts/dns/ in Deterlab.

## Conclusion

Anonymous systems help users surf the web anonymously. Some of the popular anonymous systems are TOR, Gtunnel etc. Anonymous systems can be attacked through various ways to reveal the identity of the user. In this article we discussed about the way some of the anonymous systems work and also discussed a set of attacks called Timing Analysis through which by looking at the timing of the messages flowing through the intermediate servers, it is possible for an attacker to compromise the identity of the user and figure out the path the user is using to connect to the web.

## References

1.  Timing Attacks in Low-Latency Mix Systems  
    [http://www.cs.umass.edu/~mwright/papers/levine-timing.pdf](http://www.cs.umass.edu/~mwright/papers/levine-timing.pdf)

2.  Selective Cross Correlation in Passive Timing Analysis Attacks against Low-Latency Mixes  
    [http://isec.uta.edu/mwright/papers/scc.pdf](http://isec.uta.edu/mwright/papers/scc.pdf)

3.  Timing analysis in low-latency mix networks: attacks and defenses  
    [www.cs.utexas.edu/~shmat/shmat_esorics06.ps](http://www.cs.utexas.edu/~shmat/shmat_esorics06.ps)

4.  Studying Timing Analysis on the Internet with SubRosa  
    [http://isec.uta.edu/mwright/papers/daginawala.pdf](http://isec.uta.edu/mwright/papers/daginawala.pdf)

5.  Tor: The Second-Generation Onion Router  
    [https://svn.torproject.org/svn/projects/design-paper/tor-design.pdf](https://svn.torproject.org/svn/projects/design-paper/tor-design.pdf)

6.  Preventing Active Timing Attacks in Low-Latency Anonymous Communication  
    [http://www.cs.yale.edu/homes/jf/FJS-PETS2010.pdf](http://www.cs.yale.edu/homes/jf/FJS-PETS2010.pdf)

7.  DeterLab  
    [http://www.isi.deterlab.net/index.php3](http://www.isi.deterlab.net/index.php3)

8.  PlanetLab  
    [http://www.planet-lab.org/](http://www.planet-lab.org/)

9.  Anonymity Analysis of Mix Networks Against Flow-Correlation Attacks  
    [http://www.cs.tamu.edu/academics/tr/tamu-cs-tr-2005-2-1](http://www.cs.tamu.edu/academics/tr/tamu-cs-tr-2005-2-1)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).