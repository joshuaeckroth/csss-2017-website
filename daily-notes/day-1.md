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

