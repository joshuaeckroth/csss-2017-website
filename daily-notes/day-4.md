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
    'wparson': '35.190.144.189',
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

## Password manager

Goal is to have a tool that both creates and keeps track of passwords for various sites. The passwords must be randomly generated.

```
import random
import pickle
import os.path
import getpass
from Crypto import Random
from Crypto.Hash import SHA256
from Crypto.Util import Counter
from Crypto.Cipher import AES

# goal: repeat the random character generator below 10 times,
# and put all the symbols together into one string, and print it

# generate a random password
def generate_password():
    password = ""
    for i in range(10):
        n = random.randint(33, 126)
        sym = chr(n)
        password += sym
    return password

def save_pickle(passmap, password):
    sha256 = SHA256.new()
    sha256.update(str.encode(password))
    key_hash = sha256.digest()
    file_content = pickle.dumps(passmap)
    iv = Random.new().read(AES.block_size)
    cipher = AES.new(key_hash, AES.MODE_CTR, iv, counter=Counter.new(128))
    encrypted_content = iv + cipher.encrypt(file_content)
    with open("passwords.pkl", "wb") as outfile:
        outfile.write(encrypted_content)

def load_pickle(password):
    if os.path.exists("passwords.pkl"):
        sha256 = SHA256.new()
        sha256.update(str.encode(password))
        key_hash = sha256.digest()
        with open("passwords.pkl", "rb") as infile:
            encrypted_content = infile.read()
        iv = encrypted_content[0:AES.block_size]
        encrypted_content = encrypted_content[AES.block_size:]
        cipher = AES.new(key_hash, AES.MODE_CTR, iv, counter=Counter.new(128))
        passmap = pickle.loads(cipher.decrypt(encrypted_content))
    else:
        passmap = {}
    return passmap

pickle_pass = getpass.getpass("Enter password: ")

passmap = load_pickle(pickle_pass)

while True:
    login = input("Type a existing login name or new one: ")
    if login not in passmap:
        password = generate_password()
        passmap[login] = password
        save_pickle(passmap, pickle_pass)
    print("Password is: {}".format(passmap[login]))
```

## Asymmetric encryption

GPG: GNU Privacy Guard, a tool like PGP: Pretty Good Privacy.

Example of symmetric encryption with gpg:

```
gpg -c message.txt
```

Asymmetric uses:

1. Bob wants to send Alice a message that only Alice can read.

```
gpg --recipient alice@alice.com --encrypt message.txt
```

2. Bob wants to prove that Bob wrote a message, message is not secret though.

```
gpg --clearsign --sign message.txt
```

On other side, having Bob's public key:

```
gpg --verify message.txt.asc
```

3. Bob wants to send a secret message to Alice (and only Alice), AND also prove Bob wrote it. Thus, Bob encrypts with Alice's public key, and signs with Bob's private key.

```
gpg --encrypt --sign -r alice@alice.com message.txt
```


Increase entropy with HAVEGE: "HArdware Volatile Entropy Gathering and Expansion"):

```
sudo apt-get install haveged
```

Quote about HAVEGE:

> Linux already gets very good quality random data from the
> aforementioned hardware sources, but since a headless machine
> usually has no keyboard or mouse, there is much less entropy
> generated. Disk and network I/O represent the majority of entropy
> generation sources for these machines, and these produce very
> sparse amounts of entropy. Since very few headless machines like
> servers or cloud servers/virtual machines have any sort of
> dedicated hardware RNG solution available, there exist several
> userland solutions to generate additional entropy using hardware
> interrupts from devices that are “noisier” than hard disks, like
> video cards, sound cards, etc. This once again proves to be an
> issue for servers unfortunately, as they do not commonly contain
> either one. Enter haveged. Based on the HAVEGE principle, and
> previously based on its associated library, haveged allows
> generating randomness based on variations in code execution time on
> a processor. Since it's nearly impossible for one piece of code to
> take the same exact time to execute, even in the same environment
> on the same hardware, the timing of running a single or multiple
> programs should be suitable to seed a random source. The haveged
> implementation seeds your system's random source (usually
> /dev/random) using differences in your processor's time stamp
> counter (TSC) after executing a loop repeatedly. Though this sounds
> like it should end up creating predictable data, you may be
> surprised to view the FIPS test results in the bottom of this
> article.

## Key signing

```
gpg --recv-key [key-id]
gpg --sign-key [key-id]
gpg --send-key [key-id]
```


## SSL

Critical issue: not easy to share public keys ahead of time, before going to a website. Thus, we need a service for creating "certificates" from "certificate authorities" that are default-trusted by the browser.

- Web server has two keys: public and private
- Public key gets signed by a certificate authority, after you pay them.
- You save the signed version of your public key.
- When a browser visits your site, the browser gets the public key (with signature)
  - The browser checks signature with pre-trusted certificates.
- Future communication with the server works as symmetric encryption:
  - Password is generated by browser: it creates a random number, and then encrypts that number with server's public key.
    - Thus, only the server can see that number by decrypting with its private key.
  - This random number is hashed and that hash is used for symmetric encryption (such as AES).











