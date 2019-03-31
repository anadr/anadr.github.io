---
layout: post
title: Bug Bounty Recon
subtitle: Getting Started and Performing Reconnaissance
tags: [recon, bugbounty]
---

## A Word to the Wise

Penetration testing is _hard_, so naturally, you will run into frustrating roadblocks. When you do, take a ~10 minute walk or run ([seriously](https://staciechoice1010.wordpress.com/2014/08/08/focused-vs-diffused-mode/)!) to clear your head before returning to the keyboard.

Upon return, start challenging your assumptions. Start explaining what you're trying to do and why it's working/not working. Try to force yourself to say "I _know_ that \_\_\_\_\_ _because_ \_\_\_\_\_.  

Sometimes you can't fill in the _because_. If you can't, it becomes "I _believe_ that \_\_\_\_, and I can test that hypothesis by doing \_\_\_\_\_, \_\_\_\_\_, and \_\_\_\_\_". Rinse and repeat! If you're still stuck, start challenging your _because_ statements.  

This is a [Rubber Duck Debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging) strategy I read about in a mailing list and has helped me tremendously!  

## Discovering IP Space

[Autonomous System Number](http://bgp.he.net) - IP ranges that companies have registered. If you don't know their ASN number, you can keyword search (try typing "Tesla").  
[ARIN](https://whois.arin.net/ui/query.do) - Information about registered domains and IP's. Pretty standard pentesting resource.  
[RIPE](https://apps.db.ripe.net/db-web-ui/#/fulltextsearch) - Same note as ARIN.  
[ipv4info.com](http://ipv4info.com/tools/api/) - Search for a domain to perform a reverse lookup! Try searching for cisco.com ;)  
[Shodan](https://www.shodan.io/search?query=org%3A%22Tesla+Motors%22) - Search for org:"Tesla Motors".  

## Discovering New Targets (TLD Discovery)

If the program you're working on has a wide scope, this section is important (and fun!).  
[Wikipedia](wikipedia.org) - Find the org and see what kind of acquisitions or related companies they own.  
[Crunchase Acquisitions](https://www.crunchbase.com/search/acquisitions) section - Find any recently acquired orgs. Finance market uses this for stock information so it's fairly up-to-date.

### Linked Discovery

Linked Discovery is the concept of navigating to a page and see where it links out to.  

#### Burp Spidering    

1. Turn off passive scanning
 - Scanner > Live Scanning > Live Passive Scanning > Don't scan. If this is enabled, Burp will use a ton of memory since it'll be browsing a ton of sites.
2. Set forms auto to submit (if you're feeling frisky)
 - Spider > Options > Application Login > Handle as ordinary forms 
 - Use discretion. This is NOT recommended if the page has an email form since it'll generate lots of traffic for customer. Normal login forms are fair game.
3. Set scope to adv control and use string of target name (not a normal FQDN).
 - Target > Scope > Use advanced scope control > Include in scope keyword tesla (not tesla.com - you'll miss things! This will show all links with keyword "tesla" in them).
 - Return to site map, right click and check "Show only in-scope items".
 - Note: this will also follow links in JS script files thanks to Burp's limited parsing of JS.
4. Refresh + Walk + Browse, then spider all hosts recursively by right clicking and choosing spider selected items.
 - Check spider status in Spider > Control. Could take anywhere from 5 - 30 minutes.
5. Profit (more targets!)  

# Weighted Link and Reverse Tracker Analysis

[DomLink](https://github.com/vysec/DomLink) - Tool that uses a domain name to discover organisation name and associated e-mail address to then find further associated domains. This is useful for bug bounty and red team engagements where you need to discover more domains associated with the target.  

[BuiltWith](https://chrome.google.com/webstore/detail/builtwith-technology-prof/dapjbgnjinbpoindlpdmhochffioedbn?hl=en) -  Company that manages how websites are run and what technologies they're using. Luckily, they offer browser plugins for us! This is cool to find sites that are related, in scope but not listed on the scope ;)  

[Google Dorking](https://www.exploit-db.com/google-hacking-database/): Search for "tesla © 2017" or "tesla © 2017" or whatever their copyright normally is. If they recently changed names (you should already have this info from TLD Discovery section), maybe try "tesla motors © 2017"?  

## Discovering New Targets (Subdomains)

Use both amass and subfinder. Combined, they cover about ~30 different sources.  

[Amass](https://github.com/OWASP/Amass) - The OWASP Amass tool obtains subdomain names by scraping data sources, recursive brute forcing, crawling web archives, permuting/altering names and reverse DNS sweeping.

```bash
ziom@ubuntu:~/tools/pentest/recon/amass$ cat amass.sh
#!/bin/bash
mkdir $1
touch $1/$1.txt
amass -active -d $1 |tee ~/tools/pentest/recon/amass/$1/$1.txt
```

[Subfinder](https://github.com/ice3man543/subfinder) - SubFinder is a subdomain discovery tool that discovers valid subdomains for websites by using passive online sources.  

```bash
ziom@ubuntu:~/tools/pentest/recon/subfinder$ cat subfinder.sh
#!/bin/bash
mkdir $1
touch $1/$1.txt
subfinder -d $1 |tee ~/tools/pentest/recon/subfinder/$1/$1.txt
```
	   
[Cloudflare Enum](https://github.com/mandatoryprogrammer/cloudflare_enum) - A tool that allows you to query Cloudflare DNS data.

### Subdomain Brute Forcing

Use a solid [wordlist](https://gist.github.com/anadr/08d59ef7d40da431aabcd2db8244c52c) (this is the all.txt used below).  

[massdns](https://github.com/blechschmidt/massdns) is where it's at.   Written in C: fast + efficient. Don't bother with anything else.
* Similar to subbrute, MassDNS allows you to brute force subdomains using the included subbrute.py script:  
     ```./scripts/subbrute.py lists/all.txt example.com | ./bin/massdns -r lists/resolvers.txt -t A -o S -w results.txt```  
As an additional method of reconnaissance, the ct.py script extracts subdomains from certificate transparency logs by scraping the data from [crt.sh](crt.sh):  
    ```./scripts/ct.py example.com | ./bin/massdns -r lists/resolvers.txt -t A -o S -w results.txt```  
    
[CommmonSpeak](https://github.com/pentester-io/commonspeak) - Generate your own wordlist (Subdomain data is useful, url not so much).  

[Scans.io](https://scans.io/) - Database of recent scans.  

### Auxiliary

DNSSEC/NSEC/NSEC3 Walking - similar to DNS zone transfers but for DNSSEC (see [this](https://github.com/appsecco/bugcrowd-levelup-subdomain-enumeration/blob/master/esoteric_subdomain_enumeration_techniques.pdf) for more info).  
* [nsec3map](https://github.com/anonion0/nsec3map) - see [nsec3walker.py](https://github.com/anonion0/nsec3map/blob/master/n3map/nsec3walker.py).

GitHub Recon - Use [GitRob](https://github.com/michenriksen/gitrob) to look through GitHub repos for sensitive data.

Dorking (see [this](https://www.youtube.com/watch?v=1Kg0_53ZEq8) for more info).
* ADS Key  
* Priv Pol  
* TOS  
* AWS  
* S3

## Enumerating Targets 

At this point, we found TLD, brands, and (hopefully) a bunch of subdomains/IP's. Now, we port scan all the things!

### Port Scanning

[masscan](https://github.com/robertdavidgraham/masscan) (because Nmap is _slowwww_):  
```masscan -p1-65535 -iL $TARGET_LIST --max-rate 10000 -oG $TARGET_OUTPUT```  
Problem with masscan is it will only take IP's. You can use this script to leverage subdomains and dig to automatically get IP's:  
	
```bash
#!/bin/bash
strip=$(echo $1|sed 's/https\?:\/\///')
echo ""
echo "##################################################"
host $strip
echo "##################################################"
echo ""
masscan -p1-65535 $(dig +short $strip|grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"|head -1) --max-rate 1000 |& tee $strip_scan
```

### Credential Bruteforce

Masscan (-oG) > Nmap Service Scan (-sV and -oG) > [Brutespray](https://github.com/x90skysn3k/brutespray) Credential Bruteforce

Brutespray command:  
```python brutespray.py --file nmap.gnmap -U /usr/share/wordlist/user.txt -P /usr/share/wordlist/pass.txt --threads 5 --hosts 5```

### Visual Identification

[EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness) - takes a list of domains, visits each one, takes a screenshot, and dump to folder.  It has a function to try both protocols (http and https) unlike Aquatone and HTTPScreenshot (using the --prepend-https flag).  This means you don't have to do a port scan, just supply output from amass/subfinder. An example command could be:  
```python EyeWitness.py --prepend-https -f ../domain/tesla.com.lst --all-protocols --headless```

### Wayback Enumeration

[archive.org/web](archive.org/web) - Sometimes you win with API keys, URL structures, old files, acquisitions with older code. They also have a JSON endpoint that simplifies automation.  
[Wayback Machine Downloader](https://github.com/hartator/wayback-machine-downloader) - because why not ¯\\\_(ツ)_/¯


## Credit

The original author of this content is [@jhaddix](https://twitter.com/Jhaddix). See The Bug Hunter's Methodology 3 [slides](https://docs.google.com/presentation/d/1R-3eqlt31sL7_rj2f1_vGEqqb7hcx4vxX_L7E23lJVo/edit) and [video](https://www.youtube.com/watch?v=Qw1nNPiH_Go) for reference. Additional credit goes to each developer who wrote any tool referenced in this article - see their respective GitHub pages for more information.
