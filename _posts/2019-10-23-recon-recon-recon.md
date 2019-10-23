---
layout: post
title: "Recon Recon Recon"
date: 2019-10-23
tag: [Bug bounty, Tools]

---

One of the nice things about the Infosec community is that the community is mostly generous in sharing their ideas and tools. My boss used to say "Sharing is Caring" and this really holds true with the community.

I had some time off today due to an engagement pulling out so I decided to spend the down time combining some of the recon tools made by the community that I normally use.

## Recons tools I use:
The following are some of the tools I normally use for an external PT engagements (or bug bounty programs):
* amass (who doesn't love Amass?)
* massdns (blazing fast!)
* dnsgen (mutate those subdomains)
* subjack (check those subdomain takeovers)
* Eyewitness
* gobuster

## Automation
_"If you are doing a task more than twice? Then, automate it!"_ I hear that phrase all the time, but never actually got to it. Well today's a good opportunity...

The following scripts are used to quickly find any potential low-hanging fruit on an engagement or bug bounty program:

1. subdomain_enum.sh
    * **Objective**: Enumerate as many subdomains for as possible for the target TLD
    * **Description**: The script uses amass to collect valid subdomains, followed by massdns to brute-force, and finally dnsgen to mutate a list of of subdomains and re-feed it back to massdns to brute-force again.

2. screen_enum.sh
    * **Objective**: Take screenshots of all the sites found from the subdomains. 
    * **Description**: The script uses Eyewitness to take the screenshots. Currently I am just take screenshots on port 80,443 as this tool can take sometime and I want it to be quick.

3. dir_enum.sh
    * **Objective**: Search for any interesting or sensitive directories in the enumerated subdomains.
    * **Description**: The script uses gobuster to search for interesting directories. I used to use dirsearch but found that gobuster does the job quicker especially since it is written in GO.

The three (3) scripts are then combined into a master script - "master.sh". All output is then fed to webhook on Slack so I can monitor this from the comfort of my couch. 

![subdomain_enum](/assets/img/blog/subdomain_enum.JPG)
![screen_enum](/assets/img/blog/screen_enum.JPG)
![dir_enum](/assets/img/blog/dir_enum.JPG)

To run the tool, save all the scripts in the same directory and run the following commands:

{% highlight bash %}
./master.sh domain.com
{% endhighlight %}

Simple right? Assuming that you have already got the existing tools built :)

Script can be downloaded here:
* https://github.com/Sambal0x/Recon-tools





