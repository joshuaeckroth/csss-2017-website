---
title: Day 1 Notes
layout: note
---

# Day 1 Notes

(Daily notes from class will appear here. I'll type them as we discuss in class, and then post them on this website.)

## Python examples

```
name = input("Enter your name: ")
pi = 3.141592653589
print(name + "'s favorite number is " + str(pi))
print("{}'s favorite number is {}".format(name, pi))

x = x + 1 (is the same as) x += 1
```

First program:

```
name = input("Enter your name: ")
age = int(input("Enter your age: "))
if age >= 65:
        print("You qualify for retirement benefits.")
else:
        print("You do not yet qualify for retirement benefits.")
```

Astrology example:


```
# goal: determine astrology sign for a birth date

# dates: https://en.wikipedia.org/wiki/Zodiac#Table_of_dates

import sys

name = input("Name: ")
month = int(input("Birth month (1-12): "))
if month > 12 or month < 1:
    print("Invalid month!")
    sys.exit()
day = int(input("Birth day (1-31): "))
if day > 31 or day < 1:
    print("Invalid day!")
    sys.exit()

if (month == 3 and day >= 21) or (month == 4 and day <= 20):
    sign = "Aries"
elif (month == 4 and day >= 21) or (month == 5 and day <= 21):
    sign = "Taurus"
elif (month == 5 and day >= 22) or (month == 6 and day <= 21):
    sign = "Gemini"
elif (month == 6 and day >= 22) or (month == 7 and day <= 22):
    sign = "Cancer"
elif (month == 7 and day >= 23) or (month == 8 and day <= 22):
    sign = "Leo"
elif (month == 8 and day >= 23) or (month == 9 and day <= 23):
    sign = "Virgo"
elif (month == 9 and day >= 24) or (month == 10 and day <= 23):
    sign = "Libra"
elif (month == 10 and day >= 24) or (month == 11 and day <= 22):
    sign = "Scorpio"
elif (month == 11 and day >= 23) or (month == 12 and day <= 21):
    sign = "Sagittarius"
elif (month == 12 and day >= 22) or (month == 1 and day <= 20):
    sign = "Capricorn"
elif (month == 1 and day >= 21) or (month == 2 and day <= 19):
    sign = "Aquarius"
else:
    sign = "Pisces"

print("{}'s sign is {}.".format(name, sign))
```

## Cybersecurity notes

Colors of hats:

- white hat: good hacker
- black hat: bad hacker
- gray hat: perform both white/black, mercenary/freelance

- script kiddie: someone who just grabs code from someone else and runs it


## Techniques

### Discovery

- phreaking
  - hacking phones
- phishing
  - a hacker pretends to be someone else, sends a bogus email (for example), and tries to get your login info, etc.
- port scanning
- password cracking
  - today, uses GPUs for extreme performance
- fuzzing
  - attacking a service with nonsense input until something interesting happens, then investigate further
- social engineering
  - extracting information by impersonating or engendering trust
- key logging
  - records every key press
- vulnerability enumeration
  - discovering vulnerabilities

### Attacks

- exploiting CVEs
- DOS/DDOS
  - DOS = denial of service
  - DDOS = distributed denial of service: botnet attacks all the same server at the same time
- MITM = man in the middle: watching a connection and intercepting the traffic
- Rootkits, trojans
  - these are common when you download sketchy stuff
- Viruses
  - code that automatically infects other EXE files, etc. (programs still work, but running them spreads it more)
- Cross-site scripting (XSS)
  - a website allows user-provided code (like Javascript) to run on other user's browsers
- SQL injection
  - adding your own SQL code to a URL or input box for database access
- buffer overflow
  - adding your own code so the program executes it
- ransomware
  - lock user files until they pay
  - often require payment in bitcoin

### Defenses

- Hide ports (firewalls)
- Keep software up to date (patches)
- Use Tails (Linux distribution), comes preinstall with lots of hacking tools, install on a USB drive, doesn't touch your hard drive
- Train people to detect phishing attacks
  - send phishing emails to your own employees, and then call them out
  - stay vigilant
- Choose strong passwords
- Choose correct way to hash passwords
- Proxies (hiding a service behind another service)
- Encryption
- Detect file changes to notice rootkits
- Antivirus

