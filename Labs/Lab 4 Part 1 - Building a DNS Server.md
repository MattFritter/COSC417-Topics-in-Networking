# COSC 417: Topics in Networking
# Winter 2020 - Lab 4 Part 1

In this lab, we'll begin building our own DNS server application. This application will listen for DNS requests, and then serve responses to those requests that point some domain name towards an IP address.

For the first part of this lab, we'll focus on catching and parsing inbound DNS requests. In the second part of this lab, we'll take a look at how to respond to these requests.

## Table of Contents
- [Building a Socket](#udp)
- [Listening to UDP Traffic](#listen)
- [Testing our Connection](#testing)
- [Parsing the Request](#parsing)
- [Submission](#sub)

<a name="udp"></a>
## Building a Socket

The DNS service by default will use the UDP datagram protocol over port 53. The first task we'll need to accomplish is establishing a listening connection in our application that listens for UDP traffic on that port.

We can do this using the ```socket``` library in Python. The ```socket``` library is part of the base Python libraries and allows you to handling networking tasks at a relatively low level - individual TCP packets, UDP datagrams, or streaming connections. It contains many useful functions that we will use throughout this lab.

Importing is the same as any other Python library (you shouldn't have to install anything):

```
import socket
```

We will also want to specify the IP address and port that we will be listening on as separate variables. For this lab, we'll use the IP address ```127.0.0.1```. Note that the IP is a string, while the port is an integer:

```
server_IP = "127.0.0.1"
server_port = 53
```

Next, we'll create our socket. We need to specify two things here: the type of addresses we're intending to handle with this socket, and the type of traffic we are intending to receive.

```
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

The value ```socket.AF_INET``` is a constant defined in the ```socket``` library, denoting the IPv4 addressing type. There also exists an ```AF_INET6``` for IPv6. Similarly, the ```SOCK_DGRAM``` constant is defined in ```socket``` and refers to the fact that we are dealing with UDP datagram (hence, DGRAM) traffic.

Finally, we'll need to bind our server IP address and port to the socket, telling it which address and port we'll want to listen to. Note the two sets of brackets, as the ```bind()``` function actually takes a single ```Tuple``` argument:

```
sock.bind((server_IP, server_port))
```

At this point, our socket object is ready to go, and we can begin listening to traffic at ```127.0.0.1``` on port ```53```.

<a name="listen"></a>
## Listening to UDP Traffic

Now we're ready to begin listening to UDP traffic. To do this, we can simply use the ```socket.recvfrom(int bytes)``` function on our socket. The ```recvfrom``` function takes an integer number of bytes to fetch. By default, DNS requests via UDP (we won't concern ourselves with TCP DNS requests at this time) have a maximum size of 512 bytes - so we can simply fetch 512 bytes to get our full DNS request:

```
while True:
	data, addr = sock.recvfrom(512)
```

When we call ```recvfrom```, we get two values back: our ```data```, which contains the actual message from the UDP datagram, and ```addr```, which contains information about where this request originated from. We'll need ```addr``` later when we deal with replying to DNS requests, but for now we'll focus on ```data```.

The outer ```while True``` loop just serves to run infinitely, allowing us to pull new datagrams (DNS requests) from the socket as they arrive.

<a name="testing"></a>
## Testing our Connection

In order to test that our socket connection is working, we'll need to find a way to send DNS requests to our socket. To do this in Windows, open up your connection settings, and then under connection status, click on ```Change adapter options```:

<img src="https://i.imgur.com/v39GsTs.png">

Then, right-click the adapter you are using (WiFi, Ethernet, etc) and select ```properties```. You should get another pop-up window that shows you a variety of options - scroll until you find the ```Internet Protocol Version 4 (TCP/IPv4)``` option. Select it and click the ```Properties``` button:

<img src="https://i.imgur.com/QGKwGWS.png">

Then, we'll change the ```Obtain DNS server address automatically``` option to instead use our specified DNS server address:

<img src="https://i.imgur.com/7v5t1R0.png">

Hit okay, and our DNS traffic should now be routed to our Python application. Note that you won't be able to use the internet properly until you've changed this setting back to using the automatic DNS server, as all DNS requests will essentially blackhole into our Python application.

To see if our application is working, we can simply add a print statement to our application so far, inside of the loop:

```
while True:
	data, addr = sock.recvfrom(512)
	print(data)
```

If everything has worked properly, you should see binary data being printed out for each DNS request that arrives. You can try visiting a domain in your web browser, which will result in a request with data like this:

```
b'\xbam\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x04play\x06google\x03com\x00\x00\x01\x00\x01'
```

This is the binary data payload of the UDP datagram. Note that Python will try to display ASCII characters where possible (in this example, 'google' is shown in ASCII), while un-human-readable characters will be displayed as bytes in two-digit hexadecimal format (i.e. x00 is the hexadecimal representation of 00000000 in binary).

<a name="parsing"></a>
## Parsing the Request

Now we've confirmed we can receive DNS requests with our application, now we'll need to actually parse that information.

By default, the ```data``` object is of the ```bytes``` type. This means it can be treated as an array (sliced, indexed), but it is immutable - we can read it, but cannot change it.

The ```data``` object will have two major components: a header, and a body. The header is split into six 16-bit fields, as shown below:

<img src="https://i.imgur.com/NLaWCBH.png">

The first field, the ```identifier```, is important because it identifies the DNS request. The DNS response is required to have the same identifier as the request, allowing the client to match the response to the request.

Since the identifier is 16 bits long = 2 bytes, we can get it by simply slicing from [0:2], which will fetch the first and second byte:

```
identifier = data[0:2]
```

We can do the same for the other 16-bit fields in the header. Most of these we won't worry about too much for our application - most of the control flags won't be important, and we should expect only the question count to be non-zero. We can fetch the next 16-bit field, the ```control``` field, using the following slice:

```
control = data[2:4]
```

Go ahead and get the remaining 16-bit slices for the Question count, Answer count, Authority count, and Additional count.

Once you've done that, we're finished with the head of the datagram, and we can move onto parsing the body of the datagram. Since the header is 12 bytes long (16 bits = 2 bytes * 6), the body will always start at the 13th byte in the message.

In a DNS request, the body usually consists of one or more questions. This question specifies the domain name we are trying to look up, and the type of record that we want to request.

The first field is the domain name, and it is variable-length (since domain names themselves have variable length). The domain name is split at each period, and then the length of each subcomponent is prepended as a single octet (byte), and the whole thing is ended with a zero-byte (0x00).

For example:

```
www.google.com
```

Becomes

```
www google com
```

Which when prepended with the lengths:

```
0x03www0x06google0x03com0x00
```

Since the lengths are prepended, we can read the length octet, then read the next N bytes (where N is the value of the length octet), and then continue doing this until we hit the zero byte. The code to do this is given below:

```
octet = 12
namedata = []
while data[octet] != 0x00:
	wordlen = int(data[octet]) + 1
	word = data[octet + 1:octet+wordlen]
	namedata.append(word)
	octet += wordlen
```

This will read all parts of the domain name until it reaches the end (0x00). Each domain name piece will be stored in the namedata list. Note that we start at octet 12 of the message, which is actually referencing octet 13 since Python (like most languages) uses zero-indexed arrays. This code should go inside your ```while True``` loop.

Besides the domain name, questions have two other fields that are important. These fields begin immediately after the end of the domain name. Note that going forward, we'll need to use the ```octet``` variable as our reference, since the variable length of the domain name means we don't know precisely what byte the next two fields may start on (other than as a relative reference to where the domain name ended).

The next field in the body is the ```question type```. This is a 16-bit value that specifies what type of record we are requesting. For our purposes, we'll only be looking for a value of ```0x0001```, which means an A record - the basic type of record linking a domain name to an IP address.

Since the field is 16 bits, we just need to slice the next two bits after the domain name to get the record type:

```
recordType = data[octet+1:octet+3]
```

The last part of the question body is the ```question class```. This is almost always ```0x0001```, which means a request via the Internet. Other values are used for separate, non-internet networks. This is also a 16-bit field, and can be simply sliced as before:

```
questionClass = data[octet+3:octet+5]
```

With that, we've successfully parsed a basic DNS request. Try adding print statements to print out the ```recordType```, ```namedata``` list of domain name components, and ```identifier``` for each DNS request that arrives.

Don't forget to set your DNS configuration back in the Windows settings once you are done, otherwise you won't be able to resolved domain names normally.

<a name="sub"></a>
## Submission

No submission for this lab - We'll be continuing on with this code in Part 2.
