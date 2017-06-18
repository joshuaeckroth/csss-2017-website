---
title: Day 5 Notes
layout: note
---

# Day 5 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

## Proxies

A proxy is like a fake server that stands between your machine and other machines (it "proxies" communications). One use is to filter information (filter out broken HTML pages, Javascript, ads, etc.).

A proxy makes internet requests on your behalf. So, all requests seem to come from your proxy.

Thus, if you want to hide your actual address (IP), you can use a proxy. You'll want to be sure the proxy does not record your origin address in the headers (or wherever). Of course, you might want that proxy to not trace back to you either.

## Reverse proxy

Here, the proxy lives on the server's side, and pretends to be one address. But in fact, it is taking your requests and sending them off to other places, and the replies come back out through the reverse proxy.

## Tor

The goal of Tor ("The Onion Router") is to anonymize traffic. The destination server (which hosts the stuff you want) is known, and might be shut down. But Tor allows you to hide your traffic to the server, so no one will know it was you.

## BBS Client with Tor

Download [socks.py](/socks.py)

```
import socket
import socketfuncs
import socks

domain = input("Which BBS server do you want to connect to? ")

s = socket.socket()
# connect to DNS server
s.connect(('35.185.88.75', 10334))
print("Connected to DNS server")
socketfuncs.send_message(s, domain)
print("Sent domain request to DNS server")
ip = socketfuncs.receive_message(s)
print("Got IP response: {}".format(ip))
s.close()

# connect to BBS server, through a proxy
s2 = socks.socksocket()
s2.setproxy(socks.PROXY_TYPE_SOCKS5, '127.0.0.1', 9050)

s2.connect((ip, 10333))
print("Connected")

# receive from server all prior messages
msgcount = int(socketfuncs.receive_message(s2))
print("Got count of {} messages".format(msgcount))

for i in range(msgcount):
    msg = socketfuncs.receive_message(s2)
    print(msg)

msg = input("Enter your message: ")

socketfuncs.send_message(s2, msg)

s2.close()
```

## BBS Server with GeoIP

```
import socket
import socketfuncs
import GeoIP
import re

geoip = GeoIP.open('/usr/share/GeoIP/GeoIPCity.dat', GeoIP.GEOIP_STANDARD)

s = socket.socket()
s.bind(('0.0.0.0', 10333))

msglist = []
while True:
    print("List so far: {}".format(msglist))
    s.listen(10)
    (conn, address) = s.accept()
    # can we use reverse-dns to find out the domain of this ip?
    # connec to dns server
    s2 = socket.socket()
    s2.connect(('35.185.88.75', 10334))
    socketfuncs.send_message(s2, address[0])
    domain = socketfuncs.receive_message(s2)

    print("Accepted connection from {}".format(domain))

    # determine if we still have an ip address (domain not known),
    # and figure out the geographic location

    # this needs: import re
    ipregex = re.compile(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}')
    if ipregex.match(domain):
        geoinfo = geoip.record_by_addr(domain)
        print("Geo info: {}".format(geoinfo))

    # send client all prior messages (in order)
    #print("Sending count {} to client".format(len(msglist)))
    socketfuncs.send_message(conn, str(len(msglist)))

    for i in range(len(msglist)):
        socketfuncs.send_message(conn, msglist[i])

    msg = domain + ": " + socketfuncs.receive_message(conn)
    msglist.append(msg)

s.close()
```

## Summary

### Linux summary

You can get your own Linux vm from [Google for free](https://cloud.google.com/free/).

Transfer files with [Filezilla](https://filezilla-project.org/).

### Python summary

- variables: `x = 5`; types: integer, string, boolean
- conditionals: `if x < 5:` or `elif x < 5:` and `else:`; code inside the condition must be indented
- functions: they allow you to isolate repeated code under a single name, possibly with arguments; `def myfunc(a,b):`, contents of function must be indented
- random numbers: `import random`, `random.randint(lower, upper)`
- loops: `while x < 5:` or `for i in range(10):` and `for k in mydictionary:`, allow you to repeat code until the condition is false or `break`
- lists: `vals = []`, or `vals = [1, 2, 3]`, and looping through list: `for x in vals:` and adding to list: `vals.append(4)`
- dictionaries: `d = {}`, or `d = {"key1": "val1", "key2": "val2"}`, and looping through keys of dictionary: `for k in d:` and getting value out of dictionary: `d["key2"]` and setting a value: `d["key3"] = "val3"`

Download Python 3 for [Windows](https://www.python.org/downloads/).


### Cybersecurity summary

- avoid phishing attempts
- avoid installing random apps
- use random passwords, never reuse a password, use a password manager
- use gpg/pgp to send encrypted emails
- use gpg/pgp signing to prove you authored messages or programs
- run as few services as possible, use firewalls to prevent incoming attacks and to prevent apps from making outside connections
- use Tor to stay anonymous
- only in the case of sending tips to NYTimes, use the dark web














