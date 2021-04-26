# Target Information

  **Date**         04/26/2021
  ---------------- -------------------------------------------------------------
  **Name**         Bluemoon
  **Difficulty**   Beginner
  **Location**     [Vulnhub](https://www.vulnhub.com/entry/bluemoon-2021,679/)
  **Author**       [Cyberheisen](https://www.twitter.com/cyberheisen)

# [Obligatory Disclaimer]

The tools and techniques described in this material are meant for
educational purposes. Their use on targets without obtaining prior
consent is illegal and it is your responsibility to understand and
follow any applicable local, state, and federal laws. Any liability
because of your actions is yours alone.

Any views and opinions expressed in this document are my own.

# [Walkthrough]

We start with some enumeration

 

![](images\image1.png)

 

![](images\image2.png)

 

Port, 21, 22, and 80 are open.

 

The FTP on 21 does not allow anonymous, so let\'s look at port 80.

 

![](images\image3.png)

No links and nothing hidden in the page source. Let\'s try to brute
force some directories.

 

 

![](images\image4.png)

 

And we\'ve found a directory called \'/hidden_text\'.

 

![](images\image5.png)

...it contains a QR code.

 

The decoded QR code contain FTP credentials.

![](images\image6.png)

 Let's try using them on the ftp server.

![](images\image7.png)

 

 

![](images\image8.png)

 

Two steps forward, one step back. The creds allow us to log into the ftp
server and list the files, but we\'re not able to download them.

 

Let\'s try those credentials on our other service, SSH, and see if we
can get a remote shell.

 

![](images\image9.png)

That worked; we have a shell.

 

Any user flags? Yes, we\'re able to grab the flag in Robin\'s folder.
However, we\'re not able to read the flag in Jerry\'s folder.

 

![](images\image10.png)

 

Fl4g{u5er1r34ch3d5ucc355fully}

 

Now that we\'re on the system, let\'s try viewing those ftp files again.

 

![](images\image11.png)

 

So we know that Robin likely has chosen one of these passwords. Let\'s
run this list of through crackmapexec and see if we can identify a
working password. .

 

 

![](images\image12.png)

 

 

We have a hit positive result for a valid password.

 

Let\'s try to ssh using the new credentials.

 

![](images\image13.png)
 

It worked! Do we have any sudo capabilities?

 

![](images\image14.png)

 

That\'s useful. It means we are allowed to run the feedback.sh script
with Jerry\'s permissions without a password.

 

![](images\image15.png)

 

The script Is basic. Whatever we enter as feedback is going to be
executed at the terminal. Let\'s test this theory.

 

![](images\image16.png)

 

The -u switch tells sudo to execute as the \'Jerry\' user.

![](images\image17.png)
 

Ok, that worked. We entered \'uname -a\' as our feedback and the command
was executed. With this knowledge, let\'s see if we can get a shell.

 

 

![](images\image18.png)

 

We got a shell and Jerry's flag, but the shell is lousy. Let\'s upgrade
to a better one. We\'re just going to do a simple reverse shell back to
our kali machine.

 

![](images\image19.png)

 

![](images\image20.png)
 

We now have a remote shell back to the machine as the Jerry user.

 

We upgrade the shell to make it interactive.

![](images\image21.png)
 

...and we run some of the standard privilege escalation checks. First,
are we allowed to execute any commands as sudo?

 

![](images\image22.png)
 

Nope...

 

Let\'s see what we have running as root

 

![](images\image23.png)

 

![](images\image24.png)

 

Docker sticks out. If we have a /run/docker.sock file, we may be able to
break out of the image.

 

 

![](images\image25.png)

 

It\'s there. Let\'s try to break out. We are using the technique
described here: [Docker Breakout -
HackTricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout#mounted-docker-socket)

![](images\image26.png)

 

... and we have root access and the root flag.

 

![](images\image27.png)
 

# [Conclusion]

This was a fun multi-privilege escalation box. My initial web directory
enumeration attempt wasn't successful and I spent some time looking into
the other services and other avenues before coming back to web directory
enumeration and using a larger wordlist. From that point, it was a
relatively straight forward chain of escalations. The inclusion of
Docker as a method of privilege escalation was welcomed since it's not
something I generally come across and it was a good refresher.

Much thanks to Kirthik-\[H4ck3r\] for taking the time to put this
challenge together!

# [FLAGS]

Flags are reportedly generated dynamically when the target is reset, so
the flags below will be different on each run.

  user1.txt   Fl4g{u5er1r34ch3d5ucc355fully}
  ----------- -----------------------------------
  user2.txt   Fl4g{Y0ur34ch3du53r25uc355ful1y}
  root.txt    Fl4g{r00t-H4ckTh3P14n3t0nc34g41n}

# [Commands and Tools Used]
|--|--|
  **Name**                                               **Description**                                                                                                                                                                                                                                                             **How it was used**
  ------------------------------------------------------ --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------
  [AutoRecon](https://github.com/Tib3rius/AutoRecon)     AutoRecon is a multi-threaded network reconnaissance tool which performs automated enumeration of services. It is intended as a time-saving tool for use in CTFs and other penetration testing environments (e.g. OSCP). It may also be useful in real-world engagements.   Used to do the initial enumeration discovery of the target.
  [dirb](https://tools.kali.org/web-applications/dirb)   URI and DNS Subdomains brute force tool                                                                                                                                                                                                                                     Used to brute force potential files and directories at the URI
  [Firefox](https://firefox.com)                         Web browser                                                                                                                                                                                                                                                                 Used to view the web site served on the target
