# COSC 417: Topics in Networking
# Winter 2020 - Lab 5

In this lab, we're going to be exploring the mechanics of a DNS amplification attack. You will use the ```scapy``` library to send DNS queries very similar to a DNS amplification packet, but without the spoofed source IP address.

Instead, you'll aim the traffic at a central server which will keep track of your requests. Your goal will be to maximize the amplification and DNS response size - with the chance to compete with your fellow students.

## Table of Contents
- [Using Scapy](#scapy)
- [Submitting and Tracking Scores](#scores)
- [Marking](#marking)

<a name="scapy"></a>
## Using Scapy

The ```Scapy``` library allows us to craft custom DNS packets easily. Installing the library is easy using pip:

```
pip install scapy
```

Among other things, Scapy can spoof IP addresses in source headers, as well as generate all sorts of strange, malformed, and otherwise malicious network traffic. It does however make it much easier to piece together a DNS request without resorting to working with individual bytes.

When importing, you want to use the ```scapy.all``` module with a wildcard import:

```
from scapy.all import *
```

Once imported, you can use scapy to craft packets a layer at a time, beginning with the IP packet itself:

```
myRequest = IP(dst="68.183.198.86")
```

The IP packet allows you to set ```src``` and ```dest``` IP addresses for a packet. In a normal amplification attack, the target's IP would be spoofed for the ```src``` address. For this lab, we'll leave the ```src``` alone (defaults to the real source IP), but we'll set our ```dest``` to ```68.183.198.86```, the IP of the central server.

Then, we'll add on the next layer of our packet: the UDP connectionless transport protocol. This layer is simple: we only need to set the port to the default DNS port ```53```.

```
myRequest = IP(dst="68.183.198.86")/UDP(dport=53)
```

Finally, we can add the actual DNS message to the packet. You may want to set a variety of different variables in this segment - Scapy allows manual setting of all control bits, as well as the field counts, questions, and answers.

Here is a basic DNS request, for A records associated with ```www.google.com```:

```
myRequest = IP(dst="68.183.198.86")/UDP(dport=53)/DNS(id=67890,rd=1,qd=DNSQR(qname="www.google.com", qtype=1))
```

You can then send this request (without waiting for a response) using the ```send``` function:

```
send(myRequest, verbose=1)
```

This will let you know when the message has been sent. If you want to wait for an output (for testing purposes - use an actual DNS server like Google's ```8.8.8.8``` for your ```dest```), you can use the ```sr1``` function:

```
result = sr1(myRequest, verbose=1) #Send request, wait for response
print(result) #Print the result of the request
print(len(result)) #Print the length of the result
```

Once you've got this working, try different combinations of domain requested, type of request, and control settings to try to maximize the size of your response packet. You will also want to try this against different DNS resolvers (for testing, set the ```dest``` IP to the IP of the DNS resolver, and use the ```sr1``` function).

<a name="scores"></a>
## Submitting and Tracking Scores

The central server at ```68.183.198.86``` will allow you to test DNS queries. To start, you'll need to visit the server IP in your web browser - you should see a screen that looks like this:

<img src="https://i.imgur.com/1xWS7zS.png">

Enter your student number and a nickname (keep it semi-appropriate please). Make sure you enter your student number correctly, as the database records will be used to grade you. Click create account, and you will be brought to a dashboard page:

<img src="https://i.imgur.com/P4FYtNq.png">

Note the identifier that has been generated for you. This is used to identify which DNS packets belong to you. When crafting your Scapy packets, you should use the identifier provided in the ```id``` parameter in the DNS part of our packet:

```
DNS(id=<Your ID number Here>)
```

This number is the 16-bit identifier field in our DNS request - the server will associate that particular identifier with your student number. You may also adjust which DNS resolver you wish to use - simply enter the IPv4 address of the resolver in the provided text box and hit the ```Change Resolver``` button. By default, the Google ```8.8.8.8``` DNS resolver server will be used.

You can submit requests to the DNS request server at the same IP (set as the ```dest```) at port 53. Your DNS requests will show up in the table at the right. Try refreshing the page after sending a DNS request to show the results of the new request. It will display query and response size, calculate the amplification, and also show the domain and resolver used:

<img src="https://i.imgur.com/EJfsbUB.png">

At the bottom, you can see the global table of scores. Everyone's max response length, amplification ratio, total number of requests and responses, and number of resolvers tried is shown.

** If you ever get locked out of your dashboard, you can get access again by using the same student number again. **

Prizes will be awarded to the students with the three highest amplification rates, as well as the student with the largest single response size.

<a name="marking"></a>
## Marking

You do not need to submit anything for this lab. You will be marked based on following these criteria:

*You must submit at least 10 response/requests to the server
*You must achieve a response length of at least 200 bytes (without using ```baidu.com```, as we did that example in class)
*You must try at least three different DNS resolvers

Doing this will achieve you full marks. Additional bonus marks will be given for the following criteria:

*Got an amplification ratio of at least 20
*Tried at least 10 different DNS resolvers (they must be actual resolvers)
*Produced a query with a length less than 25 and a response of at least 200 bytes.

The challenge will be available until midnight, March 23rd. **Good Luck**.
