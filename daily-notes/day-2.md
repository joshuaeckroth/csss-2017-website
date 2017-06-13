---
title: Day 2 Notes
layout: note
---

# Day 2 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

## Guessing game solution

### Non-looping

```
# goal: write a guess-the-number game. allow 3 attempts
# if too high, say "too high", if too low, say "too low"

import sys
import random

n = random.randint(1, 1000)

while True:
    guess = int(input("What is your guess? "))
    if guess == n:
        print("You win.")
        sys.exit()
    elif guess > n:
        print("Incorrect, too high!")
    else:
        print("Incorrect, too low!")
```

### Looping

```
# goal: write a guess-the-number game. allow 3 attempts
# if too high, say "too high", if too low, say "too low"

import sys
import random

while True:
    n = random.randint(1, 1000)
    print("Answer: {}".format(n))
    numguesses = 1
    while numguesses <= 10:
        guess = int(input("What is your guess? (#{}) ".format(numguesses)))
        numguesses = numguesses + 1
        if guess == n:
            print("You win.")
            playagain = input("Do you want to play again? (y/n) ")
            if playagain == "y":
                break
            else:
                sys.exit()
        elif guess > n:
            print("Incorrect, too high!")
        else:
            print("Incorrect, too low!")
    print("Starting over...")
```

## How the internet works

- messages are sent as small packets
- each message has a source IP and destination IP (e.g., 192.168.10.20)
- also, each message has a source port and destination port
- packets have to routed to their destination
  - we don't all have physical connections to every other machine

There are two main protocols:

- TCP: transmission control protocol
  - rearranges out-of-order packets, detects lost packets, etc.
  - TCP handles the port

- IP: internet protocol
  - routing, IP addresses

- UDP: user datagram protocol
  - unlike TCP, allows packets to be lost, out of order, etc.

## Simple server

```
# goal: send a message to another machine, over the internet; this is the server (listens for a message)

import socket

s = socket.socket()
s.bind(('0.0.0.0', 10333))
# wait for the client to connect
s.listen(10)
# get connection info
(conn, address) = s.accept()
print("Accepted connection from {}".format(address))

# get message size (sent by client)
msgsize = conn.recv(4)
msgsize = int(msgsize.decode())
print("Got message size: {}".format(msgsize))

# receive whole message, possibly in chunks (packets)
chunks = []
bytes_received = 0
while bytes_received < msgsize:
    chunk = conn.recv(min(msgsize - bytes_received, 2048))
    chunks.append(chunk)
    bytes_received += len(chunk)
msg = b''.join(chunks).decode()

print("Got message: {}".format(msg))

s.close()
```

### Simple client

```
# goal: send a message to another machine, over the internet; this is the client (sends the message)

import socket

s = socket.socket()
s.connect(('127.0.0.1', 10333))

msg = "Hello, this is a test."
msgsize = len(msg)

# pad size with zeros if less than 4 digits
s.send(str.encode("{:0>4d}".format(msgsize)))

# send message in chunks (packets)
totalsent = 0
while totalsent < msgsize:
    sent = s.send(msg[totalsent:].encode())
    totalsent = totalsent + sent

s.close()
```

### Socket functions

```
# goal: create a library that has functions for sending/receiving messages

# provide connection object 'conn'
def receive_message(conn):

    # get message size (sent by client)
    msgsize = conn.recv(4)
    msgsize = int(msgsize.decode())

    # receive whole message, possibly in chunks (packets)
    chunks = []
    bytes_received = 0
    while bytes_received < msgsize:
        chunk = conn.recv(min(msgsize - bytes_received, 2048))
        chunks.append(chunk)
        bytes_received += len(chunk)
    msg = b''.join(chunks).decode()

    return msg

# provide socket 's' and message 'msg'
def send_message(s, msg):

    msgsize = len(msg)

    # pad size with zeros if less than 4 digits
    s.send(str.encode("{:0>4d}".format(msgsize)))

    # send message in chunks (packets)
    totalsent = 0
    while totalsent < msgsize:
        sent = s.send(msg[totalsent:].encode())
        totalsent = totalsent + sent
```

### Simpler server

```
# goal: send a message to another machine, over the internet; this is the server (listens for a message)

import socket
import socketfuncs

s = socket.socket()
s.bind(('0.0.0.0', 10333))
# wait for the client to connect
s.listen(10)
# get connection info
(conn, address) = s.accept()
print("Accepted connection from {}".format(address))

msg = socketfuncs.receive_message(conn)

print("Got message: {}".format(msg))

s.close()
```

### Simpler client

```
# goal: send a message to another machine, over the internet; this is the client (sends the message)

import socket
import socketfuncs

s = socket.socket()
s.connect(('127.0.0.1', 10333))

msg = "Hello, this is a test."
socketfuncs.send_message(s, msg)
s.close()
```

### BBS Server

```
import socket
import socketfuncs

s = socket.socket()
s.bind(('0.0.0.0', 10333))

msglist = []
while True:
    #print("List so far: {}".format(msglist))
    s.listen(10)
    (conn, address) = s.accept()
    print("Accepted connection from {}".format(address))

    # send client all prior messages (in order)
    #print("Sending count {} to client".format(len(msglist)))
    socketfuncs.send_message(conn, str(len(msglist)))

    for i in range(len(msglist)):
        socketfuncs.send_message(conn, msglist[i])

    msg = socketfuncs.receive_message(conn)
    msglist.append(msg)

s.close()
```

### BBS Client

```
import socket
import socketfuncs

s = socket.socket()
s.connect(('127.0.0.1', 10333))

# receive from server all prior messages
msgcount = int(socketfuncs.receive_message(s))
#print("Got count of {} messages".format(msgcount))

for i in range(msgcount):
    msg = socketfuncs.receive_message(s)
    print(msg)

msg = input("Enter your message: ")

socketfuncs.send_message(s, msg)

s.close()
```
