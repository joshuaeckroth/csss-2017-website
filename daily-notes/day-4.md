---
title: Day 4 Notes
layout: note
---

# Day 4 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

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

