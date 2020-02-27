# COSC 417: Topics in Networking
# Winter 2020 - Lab 4 Part 2

For this lab, we'll finish up the DNS server software that we began working on in the previous lab. In order to do so, we'll need to handle the code for sending a DNS response to the DNS query that we parsed previously.

## Table of Contents
- [Modifying the Original Message](#modify)
- [Appending our Answer](#append)
- [Submission](#sub)

<a name="modify"></a>
## Modifying the Original Message

You may want to pull up the Topic 9 notes to reference the format of the DNS request and response. The first thing that we'll want to do is dump out the original message data as a ```bytearray``` type. This will make it easier to modify individual bytes within the message, and append new data onto the end of the message as needed.

This can be done using the following code, which should go inside your server loop, but after you've parsed the message (Part 1):

```
MESSAGE = bytearray(data)
```

The ```data``` object in this case is the original object we fetched from the UDP socket in Part 1 of the lab:

```
data, addr = sock.recvfrom(512)
```

The first modification we will need to make is to the ```QR``` value in the control field. This value is 0 for requests, but must be set to 1 for responses. Since the control field begins at the third byte (the identification field takes up the first two), we can make this change as so:

```
MESSAGE[2] = 0x80
```

For the sake of simplicity, we'll zero out all the other fields in the control field. We aren't providing recursion, nor are we using any of the special flags for this lab, so this should be fine in this use case:

```
MESSAGE[3] = 0x00
```

The last change that you'll need to make is to update the answer count header field to a value of 1 (reflection one answer in our response). **Use the two previous examples to help work out how to change the answer count.**

<a name="append"></a>
## Appending our Answer

Once we've edited our message, we can begin appending our new response answer to the message. First, we'll want to append a label to the message that points to our previous label (hostname encoded as mentioned in the notes).

This is fairly straightforward, as we assume that the label begins at octet 12, we can use that as our pointer:

```
MESSAGE.append(0xC0)
MESSAGE.append(0x0C)
```

We will then need to specify the type of record that we are returning: type 0x0001, which is an ```A``` type record, as well as a class of 0x0001 (internet request):

```
MESSAGE.append(0x00)
MESSAGE.append(0x01)
MESSAGE.append(0x00)
MESSAGE.append(0x01)
```

**Using these as an example, add the remaining fields to your response: the ```TTL``` field, the ```RDLENGTH``` field, and the ```RDATA``` field (a byte-encoded IPv4 address).** Use the Topic 9 notes as necessary to determine the length and format of these fields. You may use any IP address you wish.

You can then send your completed response back to the client using the following code:

```
sock.sendto(bytes(MESSAGE), addr)
```

Once you've done this, you can then test your DNS server application as you did in Part 1. You should now be able to go to a web URL in your browser and have that domain name point to the IP address of your choice.

<a name="sub"></a>
## Submission

When done, submit your completed DNS server code on Moodle.
