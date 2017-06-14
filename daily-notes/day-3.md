---
title: Day 3 Notes
layout: note
---

# Day 3 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

## DNS server

```
import socket
import socketfuncs

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

    domain = socketfuncs.receive_message(conn)

    if domain in domainmap:
        ip = domainmap[domain]
    else:
        ip = "0.0.0.0"

    socketfuncs.send_message(conn, ip)
```

Updated bbsclient.py:

```
# goal: send a message to another machine, over the internet; this is the cl
ient (sends the message)

import socket
import socketfuncs

s = socket.socket()
# connect to DNS server
s.connect(('35.185.88.75', 10334))
print("Connected to DNS server")
socketfuncs.send_message(s, 'underpants')
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

## Hashes vs. encryption

Hash:

- an IRREVERSIBLE string that was computed from a message
- there is no "key", the original message cannot be discovered

Encryption:

- a REVERSIBLE string that was computed from a message AND a key (password)
- symmetric vs. asymmetric ciphers:
  - symmetric: there is a password (key) that encrypts and decrypts the message
  - asymmetric: there are two passwords (two keys), one encrypts, other decrypts


## Passwords

Suppose you need to keep passwords in a database (with usernames) to figure out if the person typed their password correctly. How should you store that password in the database?

- plaintext (just put it in there as-is)
- hash it (yup)
  - but why? You want to store them in a way that nobody can (easily) find out what the original password was and log in as somebody else
  - but of course, you need to be able to check if a user typed their password correctly when logging in

Suppose you had only digits:

- 6 length: 10^6: 1,000,000
  - <1sec
- 10 length: 10^10
  - <1sec

Suppose you had only lower-case letters:

- 6 length: 26^6: 308,915,776
  - <1sec
- 10 length: 26^10: 141,167,095,653,376
  - 44min

Suppose you had only upper/lower-case letters and numbers:

- 6 length: 62^6: 56,800,235,584
  - 1sec
- 10 length: 62^10: 839,299,365,868,340,224
  - 181days

Suppose you have upper/lower-case letters and numbers and symbols (32):

- 6 length: (52+32+10)^6: 689,869,781,056
  - 13sec
- 10 length: (52+32+10)^10: 53,861,511,409,489,970,176
  - 32years


Example hashcat usage:

```
hashcat -w 3 --status -o output.txt --outfile-autohex-disable -m 0 -a 0 md5hashes.txt ../rockyou.txt -r /usr/share/doc/hashcat/rules/dive.rule
```

## Symmetric cipher

```
# goal: use a symmetric cipher (AES)

# requires: pip install pycrypto

from Crypto import Random
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util import Counter

msg = input("Enter a message: ")
password = input("Enter a password: ")

sha256 = SHA256.new()
sha256.update(str.encode(password))
key_hash = sha256.digest()
key_hash_printable = sha256.hexdigest()
print("Hash of key: {}".format(key_hash_printable))

iv = Random.new().read(AES.block_size) # create random IV (initialization vector)
print("Randomly generated IV: {}".format(iv))
cipher = AES.new(key_hash, AES.MODE_CTR, iv, counter=Counter.new(128))
msg_enc = iv + cipher.encrypt(msg.encode()) # prepend IV
print("Encrypted: {}".format(msg_enc))

# ... save msg_enc, remember password ...

# extract IV (assuming we don't have the variable from earlier)
iv = msg_enc[0:16]
msg_enc = msg_enc[16:]
# recreate cipher object, won't work if we reuse prior
cipher2 = AES.new(key_hash, AES.MODE_CTR, iv, counter=Counter.new(128))
msg_dec = cipher2.decrypt(msg_enc)
print("Decrypted: {}".format(msg_dec))
msg_dec_utf8 = msg_dec.decode('utf-8')
print("Decrypted: {}".format(msg_dec_utf8))
```

## Questions for IT tour

- How big is the room with the servers?
- How much RAM does each server have?
- Do you have GPUs?
- How many cores?
- How much does it all cost?
- What is the clock speed?
- What if everything crashes?
- What would happen if you had to overclock the system?
- Does it have water cooling?
- Does it have vapor chambers?
- What is the highest temperature that the room has reached?
- What temp do the CPUs idle at?
- How much storage do you have?
- Can it run Crysis?
- What happens if this machine (the one I'm pointing at) suddenly disappears?
- What if you lost internet connection to the outside world?
- What is the biggest security breach you've had?
- Which communications between machines are encrypted?
- What kinds of hashes do you store? Are they MD5?
- How do you handle backups? How long do you keep backups?
- Is there night security?
- What happens if the power goes out in DeLand?
- Are there security cameras? Does anyone watch them? Do you keep a history? Where are they?
- How has this room evolved over time?
- What programs/usage stresses the system most, in terms of CPU and network?
- Which OS's are you running?
- Do you have benchmark scores?
- How many emails are sent per day?
- Does anyone encrypt their email?
- How many wifi spots do you have?
- How flexible is the system for different kinds of uses?
- How does it compare to other schools' systems?
- How much does this kind of job pay?
- What are the qualifications for this kind of job?
- Are you hiring?
- Can students intern at IT? e.g., work study?
- Are all CPUs identical or is there a mix?
- What do you do with old servers?
- Do you still have the oldest server?
- Do you support VOIP (voice over IP), webcam access, etc.?
- How do you identify a broken server?
- How many employees do you have and what do they do?














