---
layout: post
title: "Hacking Razer Pay Ewallet App"
date: 2020-04-20
tag: [Bug bounty, Mobile]
---

##Introduction
This write-up is about hacking the Razer Pay Android app - an E-Wallet app used in Singapore and Malaysia. It was an interesting journey worth blogging due to the user of some interesting techniques including Frida, a tool that I only thought was meant for bypassing SSL-pinning or root detection. In this write-up I will show how I was able to use Frida to compromise the app, ranging from reading other user's chat messages, deleting user's bank accounts, gleaning user's private info, and even stealing money from user's accounts.


##TL;DR
The Razer Pay app utilised signatures to prevent tampering of request payloads. Each GET and POST request that was delivered to the server was protected with a calculated signature field. 

![Razer-Pay1...](/assets/img/blog/Razer-Pay1.png)

Any attempts to modify the request parameters would result in a error response. However by reverse engineering the APK file and identifying the right methods, it was possible to to calculate the new signatures for modified payloads. Combining the use of Frida to automate the re-calculation of the new signatures, it was possible identify a large number of IDOR issues.

##Background
When selecting programs to participate on, I normally like to choose programs that have higher barrier to entry to avoid duplicates. When Razer decided to participate on the Hackerone platform, the Razer Pay app caught my eye. This was because a Malaysian or Singaporean mobile number was required for registration. Living in Australia, I did not own a Malaysian/Singapore mobile numbers. However I had a some friends and family members who lived in Malaysia that were willing help source some Malaysian numbers for testing. Knowing that many bug bounty hunters would mostly have this resistance, I decided to take a deep dive in this app :)

![Razer-Pay1...](/assets/img/blog/Razer-Pay2.png)


I started analyising this application by proxying traffic using Burp and quickly noticed that a lot of parameters could not be tampered. This is due to the signature field in each request. My first thought was to register another user account and copy the same payload for an action that could only be made by that user, and perform it on the first user's session. But that failed as the signature was also calculated based on the second user's session auth header.

I was determined to find how the signatures were calculated. So I decompiled the application using **apktool** and use **Jadx-Gui** to understand how the application worked. I started to search for API endpoints and traced it back a method called "MD5Encode". The name suggested "MD5 hash" algorithm but I was never able to calculate the right signature by MD5-ing the payload (perhaps they were in the wrong order or wasn't the normal implementation of MD5).

![Razer-Pay3...](/assets/img/blog/Razer-Pay3.png)

##Deleting other user's bank account
Not giving up, I decided to copy all the Java code and perform the signature calculation offline. My choice of IDE was **IntelliJ IDEA**. At this point, I knew the the right paramter value sequence to insert into the "MD5Encode" method based on my previous source code analysis. There were some minor tweaking required to get the code to compile (due to the code obfuscation) but it wasn't too hard. Placing in the same parameter values for an existing request I previously made into the method yielded the same signature, so I knew I was on the right track!

![Example...](/assets/img/blog/RazerPay4.png)
![Example...](/assets/img/blog/RazerPay5.png)

I wanted to test for IDORs as I thought there is a good probability that developers were depending on the `signatures` to protect against parameter value tampering. In this specific API that I was testing `/deleteBankAccount`, I calculated the signatures for a range of predictable `id` values and its corresponding `signature` value. I then selected the `id` (bank Id) for a second account that I owned and delivered the payload.

This worked and I was able to delete my another user's bank account! Nasty...

![Example...](/assets/img/blog/RazerPay6.png)


##Frida to the rescue
At this point, I knew there must be other API endpoints that vulnerable IDOR but protected by the signature field. I tried to repeat the same attack I did previously but nothing worked. This was because other API endpoints used different algorithm and were painful obfuscated. The code compilation failed and I was wasting a lot of time troubleshooting obfuscated code.

This is where Frida came to the rescue. Frida was awesome as it did all the heavy lifting and all I needed to do was identify the right method that I wanted to hook. By hooking the right method and providing Frida with the neccessary values it needed to calculated the new signatures, I was able to quickly automate things and get the right signature.

```
// frida.js - Use this for recalculating signature for adding user to other people's chatgroup

console.log("Starting...")
Java.perform(function () {
    var MD5 = Java.use('com.mol.molwallet.view.MD5')
    MD5.MD5Encode.implementation = function (arg)
    {
        console.log("Hooking class MD5 - method MD5Encode")

       //Extra step - calculate new signature
        var ret_value = this.MD5Encode("groupId=1x9&userIds=95xxx7&token=b6fxxxd3-2xxc-4xxf-bxx7-7fxxxxa6")
        console.log("[+]  signature= " + ret_value)

        //Call method with original arguments so app doesn't crash ..
        var ret_value = this.MD5Encode(arg) //original value
                console.log("original ARG: " + arg)  
        return ret_value;
    }
})
```

To run the frida, I had to have root access (This did not matter as I was testing a server side issue and an attacker could easily use a rooted device to perform the attack). The following commands were used to start frida server on the mobile device.

```
$ adb shell
# sudo su
# /data/local/tmp/frida-server
```

On a different terminal tab, I ran the following commands to run the Frida script.

```
$ frida -l frida.js -U com.mol.molwallet
```

As I browsed the mobile app, any action that utilised the hooked method "MD5Encode" would run the script loaded. As a result, I was able to get the right signature that I needed to make a valid request. In this specific example, I was able to add myself to a chatgroup that I was not invited to. The impact of this is that I am able to view other user's chat messages , and worst of all steal unclaimed money that is shared betweem members in a group chat.

![Example...](/assets/img/blog/RazerPay7.png)
![Example...](/assets/img/blog/RazerPay8.png)


##Rinse and repeat
I repeated the same technique with Frida for all other API endpoints that were vulnerable. Some high impact issues include enumerating how much red packet (money) was shared between users or in a group, viewing other user's transaction details and personal information.


##Closing
The total bounty received for reporting these issues was approximately $6,000, a little low considering the number of finding reported including the impact that the vulnerabilities could have had on Razer's user. The disclosure and reward process with Razer team were at time frusfrating due to the lack of response. Sometimes I cringe when I read my multiple follow ups for a response or follow ups on bounty rewards that were forgotten. However most of the issues were resolved recently and I have personally decided not to carry on working for this program going forward.