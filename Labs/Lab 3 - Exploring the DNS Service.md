# COSC 417: Topics in Networking
# Winter 2020 - Lab 3

In this lab, we'll be taking a look at the DNS service using Wireshark. We'll be able to see DNS packets in-flight, and also see how caching changes how DNS works. Answer the questions in this lab in a Word document for submission.

## Table of Contents
- [Capturing DNS Traffic](#capture)
- [DNS Record Types](#dns)
- [Local Caching](#cache)
- [Submission](#sub)

<a name="capture"></a>
## Capturing DNS Traffic

We can use the Wireshark utility to capture many types of web traffic, including DNS requests. Open up Wireshark (it should be installed on the lab machines), and open up a web browser.

To start with, we'll visit some common, popular websites: ```www.google.com``` and ```www.wikipedia.org```. Hit the ```capture``` button in Wireshark, and then navigate to these two websites. Then, stop your capturing.

If you sort your captured output by protocol, and scroll up to the DNS section, you should hopefully see something like this:

<img src="https://i.imgur.com/1hahxWu.png">

**Q1: You may notice, in your results and in the above screenshot, that your DNS requests are going to a local ```192.168``` IP address. Try navigating to this IP address - what is it?**

**Q2: What does this suggest is happening with these DNS requests? Are we hitting the DNS root servers?**

<a name="dns"></a>
## DNS Record Types

Another thing we can take a look at is the contents of the DNS packets themselves. Clicking on one of the capture entries will allow you to see the contents of the captured packets. For example, this is the request packet for wikimedia:

<img src="https://i.imgur.com/xvr9Txk.png">

Note also that it says ```Response In: 72``` - this is the capture number of the responding message to the DNS request.

**Q3: Take a look at your request and response packets for ```www.wikipedia.org```. What do you notice about the format of the DNS request and response?**

Now, let's perform a slightly different capture. Instead of using Google or Wikipedia, this time try going to ```alpinelab.ca``` and ```www.alpinelab.ca``` while capturing in Wireshark.

**Q4: Looking at your output, what type of DNS records are returned from these two URLs?**

**Q5: Where did these DNS records originate from, according to Wireshark? Does this differ from when we tried querying Google or Wikipedia?**

In order to reinforce the behaviour seen in Q5, you can try visiting other websites that aren't commonly visited. Domains you own yourself are a good example (I own alpinelab.ca, for example).

Try making and capturing multiple requests to the same domain (such as alpinelab.ca). You may notice that your requests begin to go towards the same IP address identified in Q1.

**Q6: Based on the above, what does this suggest has happened? Why would the destination IP change?**

<a name="cache"></a>
## Local Caching

Begin a new capture in Wireshark. Try visiting a website. Verify that a DNS request appears in your capture, then close the website in your browser. Open a new tab, and try navigating to it again. You should notice that the second time you open it, no second DNS request is made.

Now, while still capturing, open a Windows command prompt, and enter ```ipconfig /flushdns```. You sbould see output like that below:

<img src="https://i.imgur.com/xPtBXO5.png">

This will flush the local Windows DNS Resolver cache.

**Q7: Try navigating to the same website again, while still capturing using Wireshark. What happens?**

You can see the contents of the DNS Resolver cache by going using the ```ipconfig /displaydns``` command.

**Q8: Try visiting a website like Google after you've flushed the cache. Pay attention to where the destination of the DNS request. Considering it shouldn't be locally cached anymore, what does this suggest?**

<a name="sub"></a>
## Submission

When you're done, submit the answers to your 8 questions on Moodle.
