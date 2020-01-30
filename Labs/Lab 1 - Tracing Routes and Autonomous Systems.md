# COSC 417: Topics in Networking
# Winter 2020 - Lab 1

In this lab, we'll be using the ```tracert``` utility to trace the path between our computer and an external target. Then, we'll use this information to try to determine which autonomous systems our traffic is travelling through.

## Table of Contents
- [The Tracert Utility](#tracert)
- [Determining Autonomous Systems](#das)
- [Learning More About the AS Topology](#topo)
- [Graph It](#graph)
- [Submission](#sub)

<a name="tracert"></a>
## The Tracert Utility (3 marks)

In Windows, the ```tracert``` utility can be used to determine the route between our own machine, and a specified target IP address or domain name. It allows us to visualize each 'hop' that our traffic makes across the network on it's way from origin to destination.

The command syntax is simply: ```tracert google.com``` or ```tracert 123.123.123.123``` in the command prompt. It may take a while to fully resolve.

<img src="https://i.imgur.com/xoQPbRn.png">

**Perform a trace route for each of the following domain names:**

* msu.ru 
* uni-hamburg.de
* uwaterloo.ca

These domains correspond to the Moscow State University (Russia), the University of Hamburg (Germany), and the University of Waterloo (Canada).

**Save the output of each trace route to a file called ```routes.txt```**

<a name="das"></a>
## Determining Autonomous Systems (3 mark)

Once you've got your trace routes completed, we'll perform some manual analysis on them to try to determine which Autonomous Systems each 'hop' point is associated with.

There are some complications to this. Some of your earliest addresses in the trace route will be local IP addresses (10.0.0.0, 192.168.0.0 addresses), which we can't look up. You may also notice some steps along the trace route give you a ```Request timed out``` message, and don't display any data. You may also find that some IP addresses are unannounced and can't be properly associated with an autonomous system without a lot of work.

We won't worry about this - we'll map what we can based on the information we have available to us.

Using a tool such as this one here: <a href="https://asn.cymru.com/cgi-bin/whois.cgi">Team Cymru Whois</a>, we can quickly determine what autonomous system an IP address is associated with. Using the output of your trace routes, check each IP in each trace and determine what the AS name and number are for each.

For example, you might see an IP like this in your tracert: ```205.189.32.181```, associated with Canarie, a Canadian network technology company. Searching for this IP using the above tool provides the following results:

<img src="https://i.imgur.com/2E0aqf8.png">

**Record the AS Name and Number for each IP in each tracert, if you can find an associated AS. Save this in information in a file called ```asn.txt```. Make sure you keep the ASNs organized in order that they appear for each trace route.**

There are many other tools available to find this information on the web. If you're having difficulties, try searching for a different tool - ```whois``` is a common term for this kind of lookup. You may also use the ```whois``` command on Linux/Mac machines to find this information.

<a name="topo"></a>
## Learning More about the AS Topology

Once we have AS names and numbers for our IP addresses, we can use other tools to find out more information about them. For example, we can use Hurricane Electric's online tools to learn more about a particular autonomous system.

To do this, simply go to ```https://bgp.he.net/AS2848```, replacing the ```AS2848``` with whatever AS you're attempting to look up. We can see from this page the autonomous system that ```msu.ru``` is part of:

<img src="https://i.imgur.com/6ByYhY6.png">

We can also see that AS2848 has only one peer: AS3267. Sure enough, if we look up the prior hop IP address, we can see it is associated with AS32367. We have confirmed that traffic flows from AS3267 to AS2848! We can also infer that since AS2848 is only connected to AS3267, that AS2848 is a **stub** and all traffic aimed for msu.ru comes via AS3267.

Using fairly rudimentary tools, we're able to determine some of the underlying network topology. There are no marks associated with this, but try looking up some other autonomous systems and see if you can guess what kind of autonomous system you are looking at (stub, multi-homed stub, transit AS, etc).

<a name="graph"></a>
## Graph It (2 marks)

Choose one of the three trace routes we looked at (uwaterloo, msu, or hamburg), and make a simple graph. You may use MS Paint or any other tool, although I recommend using <a href="draw.io">draw.io</a>.

Using the AS information you have obtained previously, graph the traffic from your home machine to the destination. Each node along the way should have the autonomous system number and name in it. Don't include an autonomous system twice - if multiple consecutive IP address hops share an autonomous system, then lump them all together.

If you couldn't determine the AS of a given IP address, or the trace route request timed out, simply include them as a ```?``` node in your graph.

When you are done, you should be able to follow from the origin to the destination, and each node in between is an AS that your traffic went through in transit.

**Save your graph as graph.png**

<a name="sub"></a>
## Submission

When you're done, submit your ```routes.txt```, ```asn.txt```, and ```graph.png``` files on Moodle.
