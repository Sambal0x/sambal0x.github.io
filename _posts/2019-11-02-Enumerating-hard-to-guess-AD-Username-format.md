---
layout: post
title: "Enumerating hard to guess AD username format"
date: 2019-11-1
tag: Gigs

---

I quite enjoy external Pentest, especially when the scope is large. There has been some really interesting stuff I have found in the past but in this post I wanted to share a little event that I came across...


## Background:
For external Pentest, I normally aim to get a foothold in the client's internal within the first day by exploiting weak authentication and password spraying on their public AD endpoints. This includes Lync/Skype, OWA, Citrix and other 3rd party application such as ServiceDesk, BYOD applications etc. If the client uses weak password or lacked MFA, this normally gets me in straight away and I complete the objective and can afford to look for more interesting ways into an organisation. This is great as it gives me times to identify vulnerabilities in organisation's custom app including any 0-days in 3rd party application.


## Not this time...
For this specific client, after enumerating a large number of usernames using Initstring's awesome tool `Linkedin2username` https://github.com/initstring/linkedin2username, I noticed that `Lyncsmash` https://github.com/nyxgeek/lyncsmash did not give me any hits for valid users.

After a bit of experimenting, I contacted the client to ask for their AD username format as I struggled to get any hits. The response from the client was that they use very "custom" formats that he himself is not aware how it is assigned. 

**Hmm...Crap...**


## Thinking outside the box...
A while back I managed to find an organisation's username format by searching Github. But this client did not use Github. Fortunately a collegue of mine suggested using other tools such as `FOCA` to extract metadata from company documents that might reveal such info. 

![Light bulb moment](/assets/img/blog/lighbulb.gif)

**Ligh bulb moment!**

After running FOCA https://github.com/ElevenPaths/FOCA across previous enumerated subdomain, sure enough I came across some hits revealing the company's username format, which happen to something like **QWEAB003**. Comparing the enumerated usernames, I quickly noticed that first 3 characters **QWE** are always constant, **A** is first character of the firstname, **B** is the first character of the last name, and the last 3 digits were seemed to be between 001 and 020. The last 3 digits made sense as there could be multiple users with the same initials (e.g. John Doe , James Doodoo)

I quickly wrote a basic python script to enumerate all possible username combinations.

```python
from string import ascii_uppercase

for a in ascii_uppercase:
    for b in ascii_uppercase:
        for num1 in range(3):   # from 000 to 020 to save time
            for num2 in range(10):
            	for num3 in range(10):
                	print("QWE{}{}{}{}{}".format(a,b,num1,num2,num3))


''' Sample output:
QWEAA000 
QWEAA001 
QWEAA002
...
QWEZZ018
QWEZZ019
```

I then fed this into `lyncsmash` and sure enough, I got hits and very quickly was able tp compromise some users due to weak passwords and lack of MFA, and gain access to their internal network.