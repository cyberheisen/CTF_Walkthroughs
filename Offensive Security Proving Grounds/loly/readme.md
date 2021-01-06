# Target Information

  **Date**         01/06/2021
  ---------------- --------------------------------------------------------------------------------
  **Name**         loly
  **Difficulty**   Intermediate
  **Location**     [Offensive Security Proving Grounds](https://www.offensive-security.com/labs/)
  **Author**       [Cyberheisen](https://www.twitter.com/cyberheisen)

# [Obligatory Disclaimer]{.ul}

The tools and techniques described in this material are meant for
educational purposes. Their use on targets without obtaining prior
consent is illegal and it is your responsibility to understand and
follow any applicable local, state, and federal laws. Any liability
because of your actions is yours alone.

Any views and opinions expressed in this document are my own.

# [Walkthrough]{.ul}

 Starting with an Nmap quick scan results from AutoRecon

 

![](\1image1.png){width="5.972222222222222in"
height="3.348611111111111in"}

Anything good on gobuster?

 

![](\1image2.png){width="3.348611111111111in"
height="0.6055555555555555in"}

 

If we go to /wordpress, we find a WordPress site, but the links don\'t
work as it points to a domain name loly.lc, rather than the ip address.

 

![](image3.png){width="6.311805555555556in"
height="8.110416666666667in"}

 

We can easily fix that by entering a static hostname entry in our hosts
file. We make a copy of our original host first, make the change, and
then put the updated hosts file back in /etc

 

![](image4.png){width="4.247916666666667in"
height="1.0277777777777777in"}

![](image5.png){width="5.2752285651793525in"
height="1.312020997375328in"}

 

 

Now we have a working website.

 

![](image6.png){width="8.247916666666667in"
height="4.284722222222222in"}

 

 

First thing we do is head over to /wp-admin and try logging in with the
default WordPress credentials. It fails, but hey... we tried!

 

The WordPress site looks default. We can run wpscan to look for any
vulnerabilities or configuration issues we can leverage.

 

![](image7.png){width="6.779861111111111in"
height="5.660416666666666in"}

 

Wpscan tells us we\'re looking at WordPress 5.5 with 8 known
vulnerabilities. Let\'s see if we can use any of those.

 

![](image8.png){width="6.807638888888889in"
height="2.8256944444444443in"}

 

CVE-2020-28037 was the best looking vulnerability for us, but after
researching it a bit, it didn\'t seem feasible.

 

Next step, let\'s try to enumerate users. We do this with the
\--enumerate u argument.

 

![](image9.png){width="6.807638888888889in"
height="4.238194444444445in"}

 

We found a user: loly!

 

![](image10.png){width="6.43125in" height="1.4861111111111112in"}

 

Now lets run a password brute force attack against Wordpress.

 

![](image11.png){width="9.779861111111112in"
height="3.3208333333333333in"}

 

 We found a valid password: fernando

 

![](image12.png){width="10.036805555555556in" height="1.15625in"}

 

Now we try to login to the admin dashboard using these creds... And
we\'re in!

 

![](image13.png){width="7.449305555555555in"
height="4.889583333333333in"}

 

So we know that the version of WordPress we\'re running has
vulnerabilities, but nothing we can really use to do an RCE. Let\'s look
at the plugins and see if any are vulnerable.

 

We have three plugins. I believe Hello dolly and Akismet Anti-Spam are
included with WordPress, so let\'s research AdRotate and see if there\'s
anything there.

 

![](image14.png){width="11.100694444444445in"
height="5.660416666666666in"}

 

Looking through the AdRotate settings, I find an upload file function
that may be helpful.

 

![](image15.png){width="9.972222222222221in"
height="3.7888888888888888in"}

 

It won\'t accept php, but it will accept zip, and the zip files are
automatically extracted. Let\'s see if we can upload a php shell.

 

We\'re going to try the php-reverse-shell.php file, located in
/usr/share/webshells/php folder in the kali distribution.

We update the php-reverse-shell.php code with our IP address.

 

![](image16.png){width="4.366666666666666in"
height="1.6604166666666667in"}

 

And we zip and upload.

\`Zip shell.zip php-reverse-shell.php\`

 

![](image17.png){width="2.1006944444444446in"
height="0.4861111111111111in"}

 

Let\'s get a listener going and try to grab the shell.

 

The file would have been uploaded to /wordpress/wp-content/banners/ as
per the AdRotate settings.

 

![](image18.png){width="9.633333333333333in"
height="2.0458333333333334in"}

 

 

We hit the url

 

![](image19.png){width="11.348611111111111in"
height="3.4770833333333333in"}

 

 

And we have a shell!

 

![](image20.png){width="9.15625in" height="1.6881944444444446in"}

 

Grabbed the local.txt

 

![](image21.png){width="6.743055555555555in"
height="4.972222222222222in"}

 

We're still working with a basic shell, so before we continue, let\'s
upgrade it.

 

![](image22.png){width="3.0277777777777777in"
height="1.054861111111111in"}

 

Terminal upgraded. Any SUIDs we could exploit?

 

Nope.

 

![](image23.png){width="4.183333333333334in"
height="3.2020833333333334in"}

 

Let\'s see if our kernel is vulnerable...

 

![](image24.png){width="2.9819444444444443in"
height="0.5319444444444444in"}

 

![](image25.png){width="8.981944444444444in"
height="0.7888888888888889in"}

 

Ubuntu 16.04.1 running Kernel 4.4.0-31.

A quick search on exploit-db.com leads me to CVE-2017-16995 -- Local
Privilege Escalation for Linux Kernel \< 4.13.9 Tested on Ubuntu 16.04.

We download and compile it. We'll use the same file upload method we
used to upload our shell, so we need to zip the exploit. 

 

 

![](image26.png){width="7.440277777777778in"
height="1.2847222222222223in"}

 

![](image27.png){width="6.679166666666666in"
height="0.7798611111111111in"}

 

 Now that we have it on the server, we'll make it executable and run it.

![](image28.png){width="7.990972222222222in"
height="4.688194444444444in"}

We have a shell and root!

 

![](image29.png){width="6.926388888888889in"
height="4.027777777777778in"}

 

# [Conclusion]{.ul}

This was a fun box and I thoroughly enjoyed the challenge. Obtaining
admin access into Wordpress was trivial, but finding a working method to
upload a shell took a little time. My basic Google search looking for
AdRotate exploits didn't turn up anything, so stumbling onto the upload
method while simply combing through the WordPress settings was exciting.
The upload function worked perfectly and once we had the initial shell,
it didn't take much longer to find a working privilege escalation. I
don't come across Wordpress websites often in my day to day job, so this
was a nice little refresher for me.

Many thanks to [SunCSR](https://www.vulnhub.com/author/suncsr-team,696/)
Team for the challenge!

# [FLAGS]{.ul}

Flags are reportedly generated dynamically when the target is reset, so
the flags below will be different on each run.

  local.txt   2c7434fc2ec10dfab3817484dbcbad91
  ----------- ----------------------------------
  proof.txt   1fc03387f1637b56f534d5d01ea9e4ff

# [Commands and Tools Used]{.ul}

  **Name**                                                                      **Description**                                                                                                                                                                                                                                                             **How it was used**
  ----------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------
  [AutoRecon](https://github.com/Tib3rius/AutoRecon)                            AutoRecon is a multi-threaded network reconnaissance tool which performs automated enumeration of services. It is intended as a time-saving tool for use in CTFs and other penetration testing environments (e.g. OSCP). It may also be useful in real-world engagements.   Used to do the initial enumeration discovery of the target.
  find                                                                          search for files in a directory hierarchy (Linux)                                                                                                                                                                                                                           Used to search for executables with the SUID bit enabled for privilege escalation as root.
  [gobuster](https://github.com/OJ/gobuster)                                    URI and DNS Subdomains brute force tool                                                                                                                                                                                                                                     Used as part of the [AutoRecon](https://github.com/Tib3rius/AutoRecon) script to brute force potential files and directories at the URI
  [Firefox](https://firefox.com)                                                Web browser                                                                                                                                                                                                                                                                 Used to view the web site served on the target
  [php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell)   php based reverse shell                                                                                                                                                                                                                                                     Used to establish a shell to the target.
  [searchsploit](https://www.exploit-db.com/searchsploit)                       local command line search script for exploit-db.com                                                                                                                                                                                                                         Used to obtain the privilege escalation exploit source code - 45010
  [wpscan](https://wpscan.com)                                                  Wordpress Security Scanner                                                                                                                                                                                                                                                  Used to enumerate Wordpress settings and users. Also used to brute force logins.
