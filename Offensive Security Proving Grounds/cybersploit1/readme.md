
# Target Information   
### Date|01/04/2021  
### Name|CyberSploit1  
### Difficulty|Easy  
### Location|Offensive Security Proving Grounds/Vulnhb
### Completed|[@Cyberheisen](https://www.twitter.com/cyberheisen)    

# Obligatory Disclaimer 

The tools and techniques described in this material are meant for educational purposes.  Their use on targets without obtaining prior consent is illegal and it is your responsibility to understand and follow any applicable local, state, and federal laws.  Any liability as a result of your actions are yours alone. 

Any views and opinions expressed in this document are my own. 

# Walkthrough 

We begin by executing AutoRecon , which will give us quick results to work with while continuing to run full scans in the background. 

![](images/CyberSploit1.001.png)

And will spin up a web server to give us easy access to all our files. 

![](images/CyberSploit1.002.png)

By running the webserver as a job and sending output to  /dev/null , we keep our terminal clear. With the webserver online, we now have quick access to our CTF files. 

![](images/CyberSploit1.004.png)

The initial port scan is complete.  Looks like we have two services: SSH and HTTP at 22 and 80, respectively.  Let's take a look at the web service. 

![](images/CyberSploit1.006.png)

The web page looks custom.  Let’s browse around and see if there’s anything useful to us. 

![](images/CyberSploit1.008.png)

I generally like to start by looking at the page source, especially when the site looks custom.  In this case, it’s a quick score as we find a commented Username: itsskv 

![](images/CyberSploit1.010.png)

None of the links on the page go anywhere. 

AutoRecon  has completed, so let's take a look at the  gobuster results and see if any additional web directories were found.  Eyeing the list ![](CyberSploit1.011.png)for "Status: 200 " we see there’s a robots.txt file. 

![](images/CyberSploit1.013.png)

The robots.txt file was interesting as it contained what appeared to be a  base64 encoded string. 

![](images/CyberSploit1.014.png)

We attempt to decode it at the command line and it's successful. 

![](images/CyberSploit1.015.png)We follow the YouTube link which takes us to the CTF author's YouTube page.  I poked around a little looking for clues, but it didn't seem like it was much more than a “plug” back to for his content. 

![](images/CyberSploit1.016.png)

I spent a bit more time poking around the box and hitting dead ends, so it’s time to summarize what we know: 

- We have a username 
- We have a link to a website that was encoded. 
- There is an SSH server running 
- We have not found any authentication web pages where we could use the username. 

The only thing I can think of at this point is trying the username on the SSH service and the decoded text from the robots.txt file as the password.  Let's try it. 

And it worked.  Seriously? 

![](images/CyberSploit1.017.png)

![](images/CyberSploit1.018.png)

Ok, so we grab the local.txt flag.  Now we need to find the proof.txt file.  We're operating as  itsskv , which does not have access to root, so let's see if we can elevate our privileges. 

A quick search of executables with the SUID bit enabled turns up nothing we can use. 

![](images/CyberSploit1.019.png)

Perhaps there's a kernel exploit we can use?  Let's find what version of Linux we're running.  It’s Ubuntu 12.04.5 LTS 

![](images/CyberSploit1.020.png)

We'll search on Exploit-db to see if there are any known privilege escalation vulnerabilities. 

![](images/CyberSploit1.021.png)

That third one looks like it may be useful.  We look through the code and download it to target. 

Next, we compile the code locally and execute. Boom!  Looks like we have root. 

![](images/CyberSploit1.023.png)

We browse to the root folder and we have found our flag. 

![](images/CyberSploit1.024.png)

# Conclusion 

Cybersploit1 wasn’t a terribly difficult box, but it did lead me on a little goose chase early on with the encoded robots.txt file.  Having simply thought it was a plug for the author’s site, I spent most of the 2 hours it took to complete enumerating as much as I could trying to find a login point or a password.  Once I had exhausted all my options, only then did I try the decoded text with the username.   From there, it was an easy and straightforward privilege escalation.   Lesson learned: keep tabs of what you have and don’t overlook any possible combinations that may move you forward. 

Many thanks to [Cybersploit](https://www.youtube.com/c/cybersploit) for his time putting this CTF together. 

# FLAGS 

Flags are reportedly generated dynamically when the target is reset, so the flags below will be different on each run. 

### Local.txt
B378ae90622a2799c60d4515f1057c9f 
### Proof.txt
29c9192e6787f6766277dab90fcc69d1 

# Commands and Tools Used 

|Name |Description |How it was used |
| - | - | - |
|[AutoRecon](https://github.com/Tib3rius/AutoRecon) |AutoRecon is a multi- threaded network reconnaissance tool which performs automated enumeration of services. It is intended as a time-saving tool for use in CTFs and other penetration testing environments (e.g. OSCP). It may also be useful in real- world engagements. |Used to do the initial enumeration discovery of the target. |
|base64 |Command line tool providing base64 encoding and decoding |Used to decode the ciphertext in the Robots.txt file. |
|[curl](https://curl.se/) |Command line tool and library for transferring data with URL s|Used to download exploit code to target. |
|find |search for files in a directory hierarchy (Linux) |Used to search for executables with the SUID bit enabled for privilege escalation as root. |
|[gobuster](https://github.com/OJ/gobuster) |URI and DNS Subdomains brute force tool |Used as part of the AutoRecon script to brute force potential files and directories at the URI, |
|ssh |Secure Shell |Used to log into the target |
|[Firefox](https://www.mozilla.org/en-US/firefox/new/?redirect_source=firefox-com) |Web browser |Used to view the web site served on the target |

