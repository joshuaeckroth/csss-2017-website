---
title: Day 4 Notes
layout: note
---

# Day 4 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

## DNS server with reverse-DNS

```
# goal: create a mini-dns server: using a python dictionary (in this f
ile), respond to requests on port 10334 for the ip address of various
"domains"

import socket
import socketfuncs
import re

domainmap = {
    'localhost': '127.0.0.1',
    'jeckroth': '35.185.88.75',
    'spoons': '35.185.98.2',
    'boxes': '35.190.157.40',
    'botkin': '35.185.124.70',
    'carattini': '104.196.56.253',
    'makeda': '35.185.127.123',
    'dale': '35.190.159.174',
    'sdavis': '104.196.204.191',
    'bing': '35.185.82.202',
    'ifong': '104.196.195.56',
    'sjtpepper': '104.196.45.199',
    'google': '104.196.152.236',
    'socketphunkz': '35.190.151.35',
    'ben_jam_in': '35.190.136.94',
    'poll': '104.196.9.225',
    'tylerl': '35.185.4.69',
    'vega': '104.196.132.246',
    'lan': '35.185.87.254',
    'cmeixsel': '104.196.51.69',
    'charles': '35.190.143.160',
    'brian': '35.190.149.3',
    'wparson': '35.190.144.189',                              [32/200]
    'nick': '104.196.100.25',
    'mario': '35.185.1.35',
    'anar': '35.185.84.111',
    'pcmasterrace': '104.196.150.42',
    'rileyr': '35.185.118.196',
    'seth': '35.190.142.195',
    'underpants': '35.185.45.30',
    'elisabeth': '35.185.33.171',
    'techiex': '35.190.158.220',
    'captcrunch': '35.190.128.34',
    'yahoo': '35.185.37.110',
    'kimchen': '35.185.40.203'
}

s = socket.socket()
s.bind(('0.0.0.0', 10334))

while True:
    s.listen(10)
    print("Listening for connection...")

    (conn, address) = s.accept()
    print("Accepted connection from {}".format(address))

    domain_or_ip = socketfuncs.receive_message(conn)
    ipregex = re.compile(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}')
    if ipregex.match(domain_or_ip):
        # looks like we're doing reverse dns this time
        print("Reverse DNS: {}".format(domain_or_ip))
        answer = domain_or_ip
        for domain in domainmap:
            if domainmap[domain] == domain_or_ip:
                answer = domain
        socketfuncs.send_message(conn, answer)
    else:
        print("Forward DNS: {}".format(domain_or_ip))
        # looks like we're doing forward dns this time
        if domain_or_ip in domainmap:
            ip = domainmap[domain_or_ip]
        else:
            ip = "0.0.0.0"

        socketfuncs.send_message(conn, ip)
```

## BBS server with reverse-DNS

```
# goal: send a message to another machine, over the internet; this is
the server (listens for a message)

import socket
import socketfuncs

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

    # send client all prior messages (in order)
    #print("Sending count {} to client".format(len(msglist)))
    socketfuncs.send_message(conn, str(len(msglist)))

    for i in range(len(msglist)):
        socketfuncs.send_message(conn, msglist[i])

    msg = domain + ": " + socketfuncs.receive_message(conn)
    msglist.append(msg)

s.close()
```

## BBS Client

```
# goal: send a message to another machine, over the internet; this is the client (sends the message)

import socket
import socketfuncs

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

# connect to BBS server
s2 = socket.socket()
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

