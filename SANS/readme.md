Completed Objectives Report

@cyberheisen

12/30/2020

# Contents
[Introduction 1](#introduction)

[Completed Objective Results 1](#completed-objective-results)

[1. Uncover Santa\'s Gift list 1](#uncover-santas-gift-list)

[2. Investigate S3 Bucket 2](#investigate-s3-bucket)

[3. Point-of-Sale Password Recovery 6](#point-of-sale-password-recovery)

[4. Operate the Santavator 9](#operate-the-santavator)

[5. Open HID Lock 11](#open-hid-lock)

[6. Splunk Challenge 16](#splunk-challenge)

[7. Solve the Sleigh\'s CAN-D-BUS Problem
20](#solve-the-sleighs-can-d-bus-problem)

[8. Broken Tag Generator 22](#broken-tag-generator)

[9. ARP Shenanigans 27](#arp-shenanigans)

[10. Defeat the Fingerprint Sensor 34](#defeat-the-fingerprint-sensor)

[11. Naughty/Nice List with Blockchain Investigation Part 1
35](#naughtynice-list-with-blockchain-investigation-part-1)

[12. Naughty/Nice List with Blockchain Investigation Part 2
39](#naughtynice-list-with-blockchain-investigation-part-2)

[Conclusion 50](#conclusion)

[Lessons learned 51](#lessons-learned)

[1. Documentation 51](#documentation)

[2. K.I.S.S aka "Keep It Simple Stupid"
51](#k.i.s.s-aka-keep-it-simple-stupid)

[3. Asking for Help 51](#asking-for-help)

[Comments & Suggestions 52](#comments-suggestions)

[Final Thoughts 52](#final-thoughts)

This page intentionally left blank

# Introduction

After receiving a portrait of himself from an anonymous sender -- with
the initials "J.F.S" -- Santa's been acting a bit strange. The castle is
under construction this year and a mess with random parts lying around;
terminals down, source codes buggy, and Santa doesn't seem to notice or
care. The elves need our help to get things back online and figure out
what is wrong with Santa in time to save Christmas!

This year, we played through the challenge as "Groovy".

# Completed Objective Results

## Uncover Santa\'s Gift list

*Difficulty*: 1/5

*Description*: There is a photo of Santa\'s Desk on that billboard with
his personal gift list. What gift is Santa planning on getting Josh
Wright for the holidays? Talk to Jingle Ringford at the bottom of the
mountain for advice.

*Answer*: Proxmark

*Method*: A billboard sign advertising the North Pole can be seen at the
Turnpike 7A exit. Our task was to reconstruct the image to identify the
gift for Josh Wright.

![](images\image1.png)

Figure 1.The billboard at Exit 7A.

With the image in hand, we notice that the gift list has been edited
with a 'swirl' effect, making the text unreadable. This type of effect
can be reversed using the swirl or warp effect available in most image
editors. For our purposes, we loaded the image into GIMP[^1] and used
the 'WARP Transform' tool set on "Swirl" to manipulate the area both
clockwise and counterclockwise until we were able to make out the text.
It was not a clean revert, but through multiple manipulations we were
able to make out enough of the text to get our answer. Josh Wright is
getting a Proxmark.

![](images\image2.png)

Figure 2.The words \'Wright\' and \'Proxmark\' can be made out after
manipulating the image in the affected area.

## Investigate S3 Bucket

*Difficulty*: 1/5

*Description*: When you unwrap the over-wrapped file, what text string
is inside the package? Talk to Shinny Upatree in front of the castle for
hints on this challenge.

*Answer*: North Pole: The Frostiest Place on Earth"

*Method*: Upon arriving at the North Pole, we are greeted by Shinny
Upatree who tells us they've lost the package file for the Wrapper3000
software. It's stored on an Amazon S3 bucket and we are provided with
the bucket_finder.rb Ruby script that searches publicly accessible S3
buckets using a wordlist.

![](images\image3.png)

Figure 3. Shinny Upatree and the Investigate S3 Bucket terminal.

![](images\image4.png)
Figure 4. The \'welcome\' message at the \"Investigate S3 Bucket\"
terminal

The initial run of the script with the default wordlist returned
multiple buckets, however all were secured and not accessible to us. The
file must be in another bucket.

![](images\image5.png)

Figure 5. Multiple buckets found, but access was denied.

The default wordlist contained 3 words: kringlecastle, wrapper, and
santa. Given the name of the software in question, we added wrapper3000
to the list and executed the script again. This time, we find an open S3
bucket at http://s3.amazonaws.com/wrapper30000/package.

![](images\image6.png)

Figure 6. Executing the script with the updated wordlist containing
\'wrapper3000\'. The open s3 bucket is found at
<http://s3.amazonaws.com/wrapper3000/package>

We download the file using the curl command :

'curl <http://s3.amazonaws.com/wrapper/30000/package> \> data'.

To determine the filetype, we execute 'file data', which returns with
'ASCII text, very long lines'. Viewing the file contents, we see what
appears to be BASE64 encoded text.

![](images\image7.png)

Figure 7. Downloading and identifying the file type.

Decoding using base64 and re-executing the 'file' command we learn it's
a Zip archive.

![](images\image8.png)

Figure 8. The decoded data file is a Zip archive.

Uncompressing the Zip archive, we are left with a bz2 archive. We
decompress this using bunzip2 and are left with a Tarball archive. This
pattern continued whereby the decompressed archive would lead to yet
another compressed filetype. We repeated the process of archive
identification and decompression until we were left with a final text
file which contained the answer to complete the objective: "North Pole:
The Frostiest Place on Earth"

![](images\image9.png)

Figure 9. After multiple decompressions, we arrive at our answer.

## Point-of-Sale Password Recovery

Difficulty: 1/5

Description: Help Sugarplum Mary in the Courtyard find the supervisor
password for the point-of-sale terminal. What's the password?

Answer: santapass

Method: The terminal required us to download an offline version of the
software for analysis. We spun up our Windows 10 dev virtual machine
because we *never, EVER,* execute, open, or analyze unknown files on
production machines.

![mean girls - IT\'S NEVER GONNA
HAPPEN](images\image10.jpeg)

The downloaded file was an executable called 'santa-shop.exe' and
executing the file unpacked and installed the software onto our Windows
10 dev machine.

![](images\image11.png)

Figure 10. Sugarplum Mary and the Santa Shop terminal.

![](images\image12.png)

Figure 11. Santa-Shop installed and executed on the Windows 10 dev
machine.

Generally, when doing any sort of software analysis, we want to have
monitoring tools, such as Regshot [^2]-- a simple and personal favorite
of mine, running to identify new file, registry locations, and settings
on the machines. However, in our instance we simply installed the file
and browsed through the user profile installation folders located in
%appdata%. We were lucky and quickly found the 'app.txt' configuration
file located at
'\<user\>\\AppData\\Local\\Programs\\santa-shop\\resources' that
contained our answer: 'santapass'.

![](images\image13.png)

Figure 12. app.txt file containing the supervisor password.

![](images\image14.png)

Figure 13. Unlocked Point of Sale system. The North Pole sales tax is
quite steep!

## Operate the Santavator

*Difficulty:* 2/5

*Description:* Talk to Pepper Minstix in the entryway to get some hints
about the Santavator.

*Answer:* ![](images\image15.png)

*Method:* By exploring the different rooms in the castle, we were able
to pick-up multiple parts that would be used to divert the 'magic dust'
into the correct receptacles. Some parts were also provided by
completing certain tasks, i.e., a portal object was retrieved after
fixing the vending machine (it wasn't used in our solution) and the
missing button was found in the "unpreparedness room" after unlocking
the door by completing the "Speaker UNPrep task at Bushy Evergreen's
terminal on the Talks floor.

![](images\image16.png)

Figure 14. The Santavator panel's starting state.

Arriving at the solution was an exercise in **trial and error**.

![Trial and error? Ain\'t nobody got time fo that - Ain\'t Nobody got
time fo that \| Meme
Generator](images\image17.jpeg)


The different receptacles required different amounts of magic dust to
light up making our objective funneling just enough magic into the three
separate receptacles to operate all the floors.

![](images\image18.png)


Figure 15. The Santavator Reference Key.

The green receptacle provided access to the Lobby, and Talks. Adding the
red receptacles provided access to the Workshop and the Roof. Magic dust
was required for all receptacles to unlock Santa's office, in addition
to having a successful fingerprint scan.

I ran into an issue on this challenge whereby the magic dust moved
terribly slow when viewing in a Firefox browser on our Linux virtual
machine. Even diverting the magic dust correctly in this instance did
not provide sustained amounts of magic dust into the necessary
receptacle to unlock the floors. Thankfully, moving to a Chromium based
browser corrected the issue.

Frustratingly, after solving [objective
10](#defeat-the-fingerprint-sensor) later in the game, we realized we
may have been able to use a similar method to bypass the maintenance
panel completely.

![Image result for frustrated meme \| Funny dog memes, Cute funny
animals, Funny animal
pictures](images\image19.jpeg)


## Open HID Lock

*Difficulty:* 2/5

*Description:* Open the HID lock in the Workshop. Talk to Bushy
Evergreen near the talk tracks for hints on this challenge. You may also
visit Fitzy Shortstack in the kitchen for tips.

*Answer:* We used the Proxmark3 to clone Bow Ninecandle's HID card
number 6023 and simulate the card at the locked workshop room to unlock
it and enter the room.

*Method:* While exploring we came across a Proxmark3 device which we
were able to access via our inventory.

![](images\image20.png)


Figure 16. Proxmark3 Device.

![](images\image21.png)


Figure 17. The Proxmark3 CLI.

The workshop contained a room to the left that was secured by an HID
card reader. The room was easy to miss when entering the room --
especially given the crowd of avatars parked next to it, but as you
moved closer, the camera would pivot slightly giving a cleaner view.

![](images\image22.png)


Figure 18. Locked door in workshop. This screenshot was taken after the
door was opened.

With our Proxmark3, we set out to clone as many HID cards as possible,
starting with Santa Claus. In total we were able to clone five cards,
including Santa, as not all elves had cards that were readable.

  **Elf**           **Card Number**
  ----------------- -----------------
  Noel Boetie       6000
  Santa Claus       6022
  Bow Ninecandle    6023
  Holly Evergreen   6024
  Angel Candysalt   6040

We returned to the workshop door and began simulating cards on our
Proxmark3 device. Eventually, card\# 6023 belonging to Bow Ninecandle,
an elf standing alone outside the Talk rooms, unlocked our door and we
were able to enter the room, which acted as a portal which allowed us to
pass through the portrait of Santa in the entry and take control of
Santa Claus.

![](images\image23.png)


Figure 19. Bow Ninecandle standing outside the Talk Rooms -- doing
nothing but waiting for his badge to be cloned.

![magicdust\] pm3 If hid read TAG ID: 2ee6e22føe (6823) magicdust\] pm3
Format Len: 26 bit FC: 113 Card • . 6e23
](images\image24.png)

Figure 20. Reading Bow Ninecandle\'s HID card.

![](images\image25.png)


Figure 21. Simulating Bow NineCandle\'s HID card at the HID secured
door.

![](images\image26.png)


Figure 22. The creepy Santa Portrait hanging in the entry signed by
J.F.S. Wonder who that could be? Also, Santa has an uncanny likeness to
SANS Ed Skoudis.

![](images\image27.png)


Figure 23. We now control Santa!

## Splunk Challenge

*Difficulty:* 3/5

*Description:* Access the Splunk terminal in the Great Room. What is the
name of the adversary group that Santa feared would attack Kringlecon?

*Answer:* The Lollipop Guild

*Method:* Solving this puzzle involved executing multiple Splunk
queries, watching a Splunk talk, and decoding ciphertext. Prior to
controlling Santa, the Splunk terminal in the Great room was not
available to us. As Santa, we are now able to access.

We are presented with the KringleCastle SOC page consisting of chat
sessions between Santa and his elf SOC staff and seven Splunk related
questions we need to answer before answering the Challenge Question.

+-----+-------------------+-------------------+-------------------+
| No  | Training          | Answer            | Method            |
|     | Questions         |                   |                   |
+=====+===================+===================+===================+
| 1\. | How many distinct | 13                | To get us         |
|     | MITRE ATT&CK      |                   | started, Alice    |
|     | techniques did    |                   | mentions a stored |
|     | Alice emulate?    |                   | search in Splunk. |
|     |                   |                   | We executed the   |
|     |                   |                   | search and        |
|     |                   |                   | manually counted  |
|     |                   |                   | the unique        |
|     |                   |                   | attacks run. Some |
|     |                   |                   | attacks had       |
|     |                   |                   | multiple runs     |
|     |                   |                   | depending on the  |
|     |                   |                   | target operating  |
|     |                   |                   | systems and we    |
|     |                   |                   | had to ensure we  |
|     |                   |                   | counted the       |
|     |                   |                   | multiples as a    |
|     |                   |                   | single distinct   |
|     |                   |                   | attack.           |
+-----+-------------------+-------------------+-------------------+
| 2\. | What are the      | t1059.003-main    | Using the search  |
|     | names of the two  |                   | from the first    |
|     | indexes that      | t1059.003-win     | question, we      |
|     | contain the       |                   | sorted the        |
|     | results of        |                   | results, found    |
|     | emulating         |                   | t1059.003, and    |
|     | Enterprise ATT&CK |                   | identified the    |
|     | technique         |                   | two associated    |
|     | 1059.003? (Put    |                   | indexes.          |
|     | them in           |                   |                   |
|     | alphabetical      |                   |                   |
|     | order and         |                   |                   |
|     | separate them     |                   |                   |
|     | with a space)     |                   |                   |
+-----+-------------------+-------------------+-------------------+
| 3\. | One technique     | HKEY_LOCAL_MA     | We executed:      |
|     | that Santa had us | CHINE\\SOFTWARE\\ | index=\* \"reg    |
|     | simulate deals    |                   | query\"           |
|     | with \'system     | Micros            | \"machineGuid\"   |
|     | information       | oft\\Cryptography | and browsed the   |
|     | discovery\'. What |                   | results           |
|     | is the full name  |                   |                   |
|     | of the registry   |                   |                   |
|     | key that is       |                   |                   |
|     | queried to        |                   |                   |
|     | determine the     |                   |                   |
|     | MachineGuid?      |                   |                   |
+-----+-------------------+-------------------+-------------------+
| 4\. | According to      | 202               | index=ATTACK      |
|     | events recorded   | 0-11-30T17:44:15Z | ostap \| table    |
|     | by the Splunk     |                   | \"Execution Time  |
|     | Attack Range,     |                   | \_UTC\",          |
|     | when was the      |                   | \"Technique\",    |
|     | first OSTAP       |                   | \                 |
|     | related atomic    |                   | "technique_name\" |
|     | test executed?    |                   |                   |
|     | (Please provide   |                   |                   |
|     | the alphanumeric  |                   |                   |
|     | UTC timestamp.)   |                   |                   |
+-----+-------------------+-------------------+-------------------+
| 5\. | One Atomic Red    | 3648              | A google search   |
|     | Team test         |                   | of frgnca brought |
|     | executed by the   |                   | us to the github  |
|     | Attack Range      |                   | [^7]page. From    |
|     | makes use of an   |                   | there we          |
|     | open source       |                   | identified what   |
|     | package authored  |                   | seemed the most   |
|     | by frgnca on      |                   | logical component |
|     | GitHub. According |                   | --                |
|     | to Sysmon (Event  |                   | A                 |
|     | Code 1) events in |                   | udioDeviceCmdlets |
|     | Splunk, what was  |                   | - and did a       |
|     | the ProcessId     |                   | Google search for |
|     | associated with   |                   | "Au               |
|     | the first use of  |                   | dioDeviceCmdlets" |
|     | this component?   |                   | "Atomic". The     |
|     |                   |                   | first result      |
|     |                   |                   | brought us to the |
|     |                   |                   | github [^8]page   |
|     |                   |                   | for T1123 --      |
|     |                   |                   | Audio Capture. We |
|     |                   |                   | then executed the |
|     |                   |                   | following Splunk  |
|     |                   |                   | search which      |
|     |                   |                   | provided our      |
|     |                   |                   | answer.           |
|     |                   |                   | index=t1123\*     |
|     |                   |                   | EventCode=1       |
|     |                   |                   | \*Wind            |
|     |                   |                   | owsAudioDevice-Po |
|     |                   |                   | wershell-Cmdlet\* |
|     |                   |                   | This results      |
|     |                   |                   | consisted of two  |
|     |                   |                   | events with two   |
|     |                   |                   | unique            |
|     |                   |                   | ProcessIds. We    |
|     |                   |                   | tried both and    |
|     |                   |                   | one was the       |
|     |                   |                   | correct value.    |
+-----+-------------------+-------------------+-------------------+
| 6\. | Alice ran a       | quser             | We ran an initial |
|     | simulation of an  |                   | Splunk query to   |
|     | attacker abusing  |                   | identify any bat  |
|     | Windows registry  |                   | files used:       |
|     | run keys. This    |                   | index=\* \*.bat   |
|     | technique         |                   | and from those    |
|     | leveraged a       |                   | results, we       |
|     | multi-line batch  |                   | started           |
|     | file that was     |                   | investigating the |
|     | also used by a    |                   | files from the    |
|     | few other         |                   | atomic-red-team   |
|     | techniques. What  |                   | Github [^9]listed |
|     | is the final      |                   | and ruling them   |
|     | command of this   |                   | out using "NOT"   |
|     | multi-line batch  |                   | operators in our  |
|     | file used as part |                   | search. index=\*  |
|     | of this           |                   | \*.bat NOT        |
|     | simulation?       |                   | batstartup.bat    |
|     |                   |                   | NOT               |
|     |                   |                   | pa                |
|     |                   |                   | rse_net_users.bat |
|     |                   |                   | The third file    |
|     |                   |                   | identified:       |
|     |                   |                   | 'Di               |
|     |                   |                   | scovery.bat[^10]' |
|     |                   |                   | had multiple      |
|     |                   |                   | lines and the     |
|     |                   |                   | final line in the |
|     |                   |                   | file was our      |
|     |                   |                   | answer.           |
+-----+-------------------+-------------------+-------------------+
| 7\. | According to x509 | 55FCEEBB2127      | index=\*          |
|     | certificate       |                   | vendor=Bro        |
|     | events captured   | 0D9249E86F4B      | cer               |
|     | by Zeek (formerly |                   | tificate.issuer=\ |
|     | Bro), what is the | 9DC7AA60          | "CN=win-dc-748.at |
|     | serial number of  |                   | tackrange.local\" |
|     | the TLS           |                   |                   |
|     | certificate       |                   | \| table          |
|     | assigned to the   |                   | cer               |
|     | Windows domain    |                   | tificate.issuer,c |
|     | controller in the |                   | ertificate.serial |
|     | attack range?     |                   |                   |
+-----+-------------------+-------------------+-------------------+

Once we had answered the seven questions correctly, we received an
instant message from Alice Bluebird giving us a base64 encoded
ciphertext. She does not provide the encryption mechanism, only
mentioning the text was encrypted with an old algorithm that uses a key.
A reference to RFC 7465 [^11]was made which references the RC4 Cipher.
For the key, we are told that it was used in the Splunk talk[^12].

![](images\image28.png)


Figure 24. Message from Alice Bluebird providing Ciphertext and clues

We viewed the Splunk talk and at 18:29, the speaker presents "The most
important slide" containing a possible password: "Stay Frosty"

![](images\image29.png)


Figure 25. Santa\'s favorite saying\...

Now that we have all the pieces, we head to CyberChef[^13], a web-based
string converter that allows us to pass input through multiple decoding
functions.

Doing an initial Base64 decode followed by an RC4 decode using "Stay
Frosty" as our key, we are able to fully decode the adversary name: "The
Lollipop Guild."

![](images\image30.png)


Figure 26. Base64 Decode followed by RC4 decode using our key

## Solve the Sleigh\'s CAN-D-BUS Problem

*Difficulty:* 3/5

*Description:* Jack Frost is somehow inserting malicious messages onto
the sleigh\'s CAN-D bus. We need you to exclude the malicious messages
and no others to fix the sleigh. Visit the NetWars room on the roof and
talk to Wunorse Openslae for hints.

*Answer:*

  **ID**   **Operator**   **Criterion**
  -------- -------------- ---------------
  19B      Equals         0000000F2057
  080      Contains       FF

*Method:* This is another objective only accessible to Santa*.* When
looking at Santa's Sleigh we are presented with a system dashboard
displaying multiple log codes scrolling quickly across the screen.
Because Jack has introduced malware into the system, there are bogus
codes adding noise to the sleigh's dashboard. Our task is to identify
these codes and exclude them from the dashboard logs.

Our first step was to clear the messages to the point where we could
start identifying the codes associated with the different sled
functions: Acceleration, Braking, Steering, Start, Stop, Lock, and
Unlock. To do this, we configured multiple 'generic' exclude statements
until no further commands were captured in the log.

![](images\image31.png)


Figure 27. Generic exceptions to stop collecting log data.

Once the log was silent, we began adding one code back at a time and
modifying the sled settings to understand the codes associated with
specific sled functionality.

Using this method, we identified the following codes:

  **Code**                         **Operation**
  -------------------------------- ---------------
  02A\#00FF00                      Start
  02A\#0000FF                      Stop
  244\#000003D9 -- 00000023FF      Acceleration
  19B\#000000000000                Lock
  19B\#00000F000000                Unlock
  080\#000000 -- 080\#FFFFFD       Brake
  019\#FFFFFFCF -- 019\#00000033   Steering

With this information, we were able to enter two exclusions that
successfully removed the noisy codes introduced by the malware.

![Sleigh deFrosted! Accelerator: 40 Brake: 4 Steering: 0 188 Comparison
Operator: Equals Message Criterion: 100 ID 19B 10 NESSRGE nss Ess
:soæyossesos Operator Equals Contains RPM • Exclude Criterion
OOOOOOF2057 Remove 080 Start Stop Lock Unlock
](images\image32.png)

Figure 28. Two exceptions entered to defrost the sleigh.

## Broken Tag Generator

*Difficulty:* 4/5

*Description:* Help Noel Boetie fix the Tag Generator in the Wrapping
Room. What value is in the environment variable GREETZ? Talk to Holly
Evergreen in the kitchen for help with this.

*Answer:* GREETZ=JackFrostWasHere

*Method:* Our initial step was to use and look around the web
application to identify any vulnerabilities we may be able to exploit to
extract the necessary information.

![](images\image33.png)


Figure 29. The vulnerable tag generator

We very quickly discovered an information disclosure vulnerability when
requesting a non-existent page. In this instance, we learned the name
and path for the tag generator server code. However, appending that path
to the URL did not return execute the script or return any source code.

![](images\image34.png)


Figure 30. Error message disclosing the name and path for the server
side script.

We moved our attention to potential endpoints that may provide us with a
method of passing commands to the underlying operating system. We
configured a proxy using Burp Suite and reviewed the web requests and
responses for each client action. By default, Burp does not provide all
responses, so we had to create an intercept rule to ensure we saw all
responses from the tag-generator.

![](images\image35.png)


Figure 31. Added Intercept Responses Rule. The 'OR' operator ensure we
see any responses from the kringlecastle.com domain.

During these tests, we discovered that the client application would make
a request to /image?id=\<filename\>.jpg. We fuzzed this input by
entering a random jpg file name and were able to receive a response.

![](images\image36.png)

Figure 32. Fuzzed URL request with successful image retrieval.

However, passing the script path and name did not work.

![](images\image37.png)


Figure 33. Failed to access the server-side code given the path
retrieved from the information disclosure.

We then attempted to pass the default string for testing web-based
directory traversals on Linux: '../../../../../../../../../etc/passwd'.

Bingo! We are still presented an error page in our browser window, but
our captured response in Burp shows the contents of the server's
'/etc/passwd' file.

![](images\image38.png)


Figure 34. /etc/passwd displayed in web server response after passing a
directory traversal string into the \'image?id=\' parameter.

Since we now know it is possible to traverse server folders outside the
web root directory, we should be able to view any files on the system so
long as the web service's user account has access. We also know we need
to access an environment value and we know the operating system is Linux
based (since we accessed an '/etc/passwd' file). Linux manages resources
as files, so we should be able to extract the environment variable
contents from the file associated with a specific process.

In Linux, environment variables are maintained in the "environ" 'file'
under \</proc/\<PID\>, where \<PID\> indicates the process. We will
attempt to view environment variables from the /proc/PID/1/environ,
which should contain the environment variables for the kernel (as PID 1
is always assigned to the kernel). We modify our last web request
replacing '/etc/passwd' with '/proc/1/environ'.

![](images\image39.png)


Figure 35. Web request to view environment variables for PID 1

The request is successful, and our response contains the environment
variables for the kernel, including the GREETZ variable.

![](images\image40.png)


Figure 36. Environment variables for the server kernel

## ARP Shenanigans

*Difficulty:* 4/5

*Description:* Go to the NetWars room on the roof and help Alabaster
Snowball get access back to a host using ARP. Retrieve the document at
/NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt. Who recused herself from
the vote described on the document?

*Answer:* ![](images\image41.png)


*Method:* This was another terminal that could only be accessed as
Santa. The objective was probably the most frustrating of all due to the
time limit placed on the terminal. The solution required multiple
scripts to be modified and executed concurrently in a 3 pane TMUX
window, all within the 30 minute time limit. I set myself a timer for 20
minutes to code and attempt the objective then used the remaining time
to copy my code modifications for the next go around.

We are told that Jack Frost has hijacked a host at 10.6.6.35 with custom
malware and our task is to get command line access back to this host and
retrieve the NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt document. We
are also provided some sample ARP and DNS packet capture files, some
Debian pkg files, and two python scripts for spoofing ARP and DNS
replies.

A quick tcpdump reveals an ARP request from ARP_requester, looking to
find who is winsrvdc2019. We can presume our first task is to spoof a
reply to ARP_requester as winsrvdc2019.

![](images\image42.png)


Figure 37. tcpdump showing ARP requests

We made the necessary modifications to the ARP_response.py script to
spoof an ARP reply. We will tell ARP_requester, the compromised machine,
that we are winsrvdc2019.

![](images\image43.png)


Figure 38. Packet modifications in ARP_response.py.

![](images\image44.png)


Figure 39. Added an \'infinite\' loop to continuously send ARP replies.

We run the arp_response.py script and check tcpdump. Our spoofing
attempt was successful, and our machine is now being sent DNS requests.
We'll need to modify the dns_response.py script to send our crafted DNS
responses.

![](images\image45.png)


Figure 40. The ARP request followed by our spoofed reply. We then see a
DNS request sent to our machine.

![](images\image46.png)


Figure 41. dns_response.py modifications. We added an \'infinite\' loop
to continuously send replies.

We check our tcpdump again and, well that's interesting... now we're
seeing an http request coming our way.

![](images\image47.png)


Figure 42. HTTP syn packet followed by a reset packet response.

Let's spin up a web server and look at the request. We'll use the
http.server module included with Python.

![](images\image48.png)

Figure 43. Python http.server module displaying malware http requests
for suriv_amd64.deb.

It looks like the malware is trying to download a Debian package file.
Now it makes a bit more sense why we have the Debian packages on the
terminal. Perhaps we can modify an existing package for the malware to
download and execute? The netcat package look perfect for our needs.

A quick Google search about Debian packages tells us about the
'postinst' file included in the package which is a script that runs
after installation of the package. So, if we have the malware download
the netcat package and we append a netcat reverse shell command to the
postinst file, we *should* be able to capture a remote shell to the
compromised system.

![](images\image49.png)


Figure 44. Figure 44. Extracting the netcat package, modifying the
postinst file, and rebuilding it as suriv_amd64.deb.

The malware is trying to download the software from a specific
directory, so we need to mimic the same directory structure and place
our file.

![](images\image50.png)


Figure 45. Copying the modified malware file into the proper directory
location.

Ahhhh! Time is running out, so let us get all our scripts going and see
if we can capture a reverse shell.

![](images\image51.png)


First, we need to start our ARP and DNS spoofers again. Because our
screen space is limited, we'll start them as background processes and
send any output to /dev/null. This will keep the terminal open for us to
use, if needed

![](images\image52.png)


Figure 46. Executing the spoofing scripts as background processes and
sending output to /dev/null.

With the scripts running, we setup a netcat listener for port 4444 and
start our webserver. We immediately see status code 200s on our web
server status, meaning the file has been downloaded, and shortly
thereafter we receive a connection message on our netcat listener. We
may have a shell!

![](images\image53.png)

Figure 47. Upper right: Netcat listener on port 4444 with connection
message from arp_requester. Bottom: http server responding to file
download requests.

To test our shell, we send the 'whoami' command. The response validates
we do indeed have a shell and are operating remotely in the context of
the 'jfrost' user on 10.6.6.35.

![](images\image54.png)

Figure 48. \'whoami\' command executed on the remote system. We are
running as the jfrost user.

A quick directory search reveals the file we are tasked with
downloading. Because we know netcat is available on both systems, we can
leverage netcat to download the file to our machine. We do this by
establishing another netcat listener on our local terminal which will
output the received connection to a file. Through our existing shell
connection, we will execute another netcat command telling the remote
machine to connect to our new listener and send the file.

![](images\image55.png)


Figure 49. Directory listing on remote machine.

![](images\image56.png)


Figure 50. netcat command executed on remote machine to send the
required file to our terminal.

![](images\image57.png)


Figure 51. Netcat listener command executed on our terminal. The file
was successfully transferred as we see in our local directory listing.

With the file on our terminal, we do a quick 'cat
NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt' to view the contents of
the file. Looks like we had a successful transfer!

![](images\image58.png)


Figure 52. Output of \'cat
NORTH_POLE_Land_Use_Board_Meeting_Minutes.txt'.

## Defeat the Fingerprint Sensor

*Difficulty:* 3/5

*Description:* Bypass the Santavator fingerprint sensor. Enter Santa\'s
office without Santa\'s fingerprint.

*Answer:* Using the browser developer tools, we modified the app.js
script containing the Santavator logic, specifically an 'IF' statement
which checked whether the Santavator's button for Santa's office was
powered *[and]{.ul}* whether the passenger was Santa. We replaced the
AND operator with OR so that the check for Santa became irrelevant to
the IF statement.

Method: Once in the Santavator, we opened the browser development tools
and began looking through the code logic to identify any possibilities
to bypass the fingerprint check. A search for 'Santa' led us to the
app.js file which contained logic to test whether the passenger was
Santa.

![](images\image59.png)


Figure 53. The original \'Santa check\' code.

![](images\image60.png)


Figure 54. The updated \'Santa check\' code. The \'OR\' operator makes
it irrelevant

We change the && or AND logical operator in the IF statement to \|\| or
OR. The modification makes the entire IF statement True so long as we
have the Santavator button powered (by the magic dust).

The change was successful, and we were sent to Santa's Office, much to
the chagrin of the elf in the room who tells us we're not supposed to be
there. This method of bypass only works because the Santavator code is
executed in our browser, so we have full access to modify the code. Had
it been hosted on the server; it would not have been possible with this
method.

## Naughty/Nice List with Blockchain Investigation Part 1

*Difficulty:* 5/5

*Description:* Even though the chunk of the blockchain that you have
ends with block 129996, can you predict the nonce for block 130000? Talk
to Tangle Coalbox in the Speaker UNpreparedness Room for tips on
prediction and Tinsel Upatree for more tips and tools. (Enter just the
16-character hex value of the nonce)

*Answer:* 57066318F32F729D

*Method:* We were given a python script to interact with the
blockchain.dat file from Santa's desk. The python script, called
naughty_nice.py was very well commented and contained detailed
information regarding the blockchain and the purposed for each function
in the script.

Just recently having finished the Snowball Fight Game task in the
Unpreparedness room, where we learned about predictability in Python's
implementation of rand(), it seemed reasonable that the exploiting the
flaw in Python's random value generator was likely the method to use for
predicting the future nonce value.

Being able to predict future random values without knowing the seed
works because...**MATH**.

![](images\image61.jpeg)


It's well above my head, but essentially if we have 624 known random
values from any given seed produced through rand(), we can do a little
math and predict all future values. Thankfully, we have a script
available to do just that -- the Mersenne Twister Predictor[^14].

But first we need to make some slight modifications to the
naughty_nice.py script. We added an array value to capture our nonces
and then a loop to iterate through each block and dump our nonce values.
A function called "dump_nonces" was also created to facilitate writing
the nonces to a file.

![- documents this is the pid, or \"person id, \" that the block is
about this is the rid, or \"reporter id, \" of the reporting elf \# this
is the Naughty/Nice score of the report \# this indicates whether the
report is about n ughty or nice behavior cl. add block(block data) print
(cl. blocks print (\'C 1: Block chain verify: % (cl. verify_chain(public
key)) ) def dump -s(filename, • -s): with open(filename, \'w\') as f:
nonce -s: f.write( \"%i: % (y, i)) print (\' • •s dumped as: %
(filename)) \# Note: This is how you would load and verify a blockchain
contained in a file c I led blockchain.dat with open(\'official public.
pem\' , \'rb\') as fh: official public key = RSA. importKey(fh. read())
c2 = Chain(load=True, filename=\'blockchain.dat\' ) print( \'C2: Block
chain verify: % (c2.verify chain(official public key)) ) nonce \#for p
in range(4): new block = c2.add block (block data) print (c2.blocks)
length of chain = len(c2.blocks) for i in range(length of chain -s .
append ( hex(c2 . blocks \[i\] nonce nonce block block block block block
data \[ data \[ data \[ data \[ data \[ \'documents\'\] - \'pid \] - \'
--- 123 \# \'rid \] - \' -456 \# \'score \] - \'sign\'\] - - Nice dump •
-s(\" -s) -s.out\", • nonce blocks)) print(c2 . blocks \[len (c2 .
blocks) \#c2 . blocks \[block\] . dump_doc (blck) \#for blolck in
range(length of chain . blocks \[block\] . dump_doc(l) 1):
](images\image62.png)

Figure 55. naughty_nice.py code modifications.

Our output contained a list of nonces and their block numbers. The block
numbers were used for reference and removed prior to loading into the
Mersenne Twister Predictor code.

![naughty---nice.py X nonces.out X 1513 1513: 17190084991913927886 1514
1514: 13779296586324663969 1515 1515: 15387878553874792199 1516 1516:
17470995668915260055 1517 1517 : 3482091951633063249 1518 1518:
10563302710829741618 1519 1519: 13919668190004450198 1520 1520:
11559549402574126074 1521 1521: 7540528952072997734 1522 1522 :
13081324064896018728 1523 1523: 5016541115489936468 1524 1524:
2457307383423950361 1525 1525: 16261321797825841404 1526 1526:
16752084358068633911 1527 1527: 69606138534615275 1528 1528:
15581810439559631622 1529 1529: 17271236807222679701 1530 1530:
16289873471365621063 1531 1531: 17220429839264193288 1532 1532:
268861252614364057 1533 1533: 17134920789160580509 1534 1534:
12968379088029033755 1535 1535: 1417767387910103304 1536 1536:
2538543693281192206 1537 1537 : 15896979806300748684 1538 1538:
14120612108965687512 1539 1539: 12584682685351616622 1540 1540:
9757567714176531656 1541 1541: 15788575260498756374 1542 1542 :
18132891387785279449 1543 1543: 5643972521975276755 1544 1544:
12288628311000202778 1545 1545 : 14033042245096512311 1546 1546:
9999237799707722025 1547 1547: 7556872674124112955 1548 1548:
16969683986178983974 ](images\image63.png)


Figure 56. Snippet of nonces.out file

And then we ran into a problem using the Twister Predictor code. The
python code provided only supports 32bit hashes. Because our nonces are
64bit, modifications are needed to use the code.

There was a code snippet on the readme.md for Twister Predictor which
seemed to indicate 64bit values could be passed to the library with some
minor changes. We modified sample code and ran our nonce values through
it.

![import import import context Iib and om sys from mCIYY37predicCor arg
f sys . stdin imEM•rt MTIgg37PredicCor predictor = ( ) for in range
(624) : x = (int (next (argf))) predictor . set randbits (x, 64) ---
with context lib. suppress (BrokenPipeError) : whi le True: print
(predictor . geCrandbiCs (64) )
](images\image64.png)


Figure 57. Modified predictor code to accept 64bit values and output
predictions.

Our modifications were successful, and we were presented with a list
future nonces.

![](images\image65.png)


Figure 58. Snippet of 64bit predicted nonces output from our modified
script.

The nonce on the last block of the blockchain was 16969683986178983974
which was associated with block \#129996. So, we count forward four
additional nonces and arrive at what should be the predicted nonce value
for block\#130000, which it is.

## Naughty/Nice List with Blockchain Investigation Part 2

*Difficulty:* 5/5

*Description:* The SHA256 of Jack\'s altered block is:
58a3b9335a6ceb0234c12d35a0564c4e f0e90152d0eb2ce2082383b38028a90f. If
you\'re clever, you can recreate the original version of that block by
changing the values of only 4 bytes. Once you\'ve recreated the original
block, what is the SHA256 of that block?

*Answer:*
fff054f33c2134e0230efb29dad515064ac97aa8c68d33c58c01213a0d408afb

*Method:* Because we're provided with a SHA256 hash for the bad block we
need to find, the first thing we needed to do was modify the
naughty_nice.py program provide SHA256 hashing rather than MD5. We
further modified the code to hash each block until we found one that
matched the known altered block SHA256 hash.

![](images\image66.png)


Figure 59. Importing the SHA256 hash library.

![def full hash(self): \#hash obj - - MD5. new() hash obj = SHA256.new()
hash obj . update(self . block data signed()) return hash obj
.hexdigest() ](images\image67.png)


Figure 60. Modification of existing full_hash function to provide SHA256
hashing.

![\#nonces - \#for p in range(4): new block = c2.add block (block data)
\#print (c2. blocks length of chain = len(c2.blocks) for i in
range(length of chain) : if (c2.blocks\[i\] . full hash() == print
(\"Jack\'s Modified Block - Block \#: % i) print (c2.blocks\[i\])
c2.blocks\[i\].. ](images\image68.png)


Figure 61. Modified code to identify the block modified by Jack.

![](images\image69.png)


Figure 62. Output of modified script showing Block \#1010 as the altered
block.

With the block number in hand: 1010, we proceed to print the block and
identified the block location as 129459 or 001f9b3 in hex. We used this
value to find the altered block within the blockchain.dat file in our
hexeditor.

![](images\image70.png)


Figure 63. The start of block 12959 within the blockchain.

Now we need to figure out what bytes need to be changed. I recalled a
conversation with the elf in Santa's office who was telling us how Jack
record showed his status as nice with an exceptionally large "niceness
score" and positive feedback from witnesses. Obviously, we'll want to
change the nice to naughty, and we'll need to do something about all
those good comments which were somehow faked.

Using the naughty_nice.py script, I exported the pdf file from Jack's
modified block. As the elf had said, there were multiple comments about
how good Jack was. They seemed a bit repetitive, but I wasn't sure how
changing 4 bits would revert this.

![C O File
lhome/parrot/Downloads/OffcialNaughtyNiceBlockchainEducationPack/129459.„
: Apps Q Debian.org Q Latest News Q Help Holiday Hack C\... \"Jack Frost
is the kindest, bravest, warmest, most wonderful being I\'ve ever known
in my life.\" --- Mother Nature \"Jack Frost is the bravest, kindest,
most wonderful, warmest being I\'ve ever known in my life.\" --- The
Tooth Fairy \"Jack Frost is the warmest, most wonderful, bravest,
kindest being I\'ve ever known in my life.\" --- Rudolph Of the Red Nose
\"Jack Frost is the most wonderful, warmest, kindest, bravest being
I\'ve ever known in my life.\" --- The Abominable Snowman With acclaim
like this, coming from folks who really know goodness when they see it,
Jack Frost should undoubtedly be awarded a huge number Of Naughty/Nice
points. Shinny Upatree 3/24/2020
](images\image71.png)

Figure 64. Jack Frost\'s altered letter showing nothing but accolades.

One of the hints we received for this objective came in the form of a
229 page slide deck [^15] regarding MD5 hash collisions. It was a little
bit of information overload, but I found some information towards the
end about hash collisions in PDF files specifically. Because PDF file
structures have a tree structure, it's possible to modify the links in
the tree to point to different areas of the PDF, potentially hiding
content. This was on page 194 in the section called UNICOLL based
exploits. The example showed the author change a single bit in a PDF
Pages object to point at page 3 rather than 2.

![](images\image72.png)


Figure 65. Modifying a PDF file through a single bit change.

I looked for the 'Pages" value of Jack's PDF and found it right next to
the phrase "\_Go_Away/Santa" which apparently Jack was using as a prefix
for the exploit. This had to be the bit.

![](images\image73.png)


Figure 66. The bit to be changed.

![](images\image74.png)


Figure 67. The changed bit after adding +1.

I added 1 to the bit and re-exported the PDF file. It worked! Opening
the pdf file now showed the original negative pdf for Jack.

![](images\image75.png)


Figure 68. The original PDF file for Jack. Shinny Upatree is far too
trusting.

The UNICOLL hash collision attack allows you to add a bit at the 10^th^
character of your prefix (Jack's go away Santa comment in this
instance), but you need to offset the change by removing a bit from the
10^th^ character in the following block. If you do this correctly, the
result will be two different files sharing the same MD5 hash. I made the
necessary changes.

![](images\image76.png)


Figure 69. Offset bit before +1 change.

![](images\image77.png)


Figure 70. Offset bit after -1 change.

That should take care of the PDF file. I used 2 bits so we need change 2
more and they'll likely need to be made on the block data itself.
Because UNICOLL requires two bit change and I have 2 bit changes left,
one bit will be the actual change while the other will be used to offset
the change.

I modified naughty_nice.py to show me the other block values the elf had
previously mentioned Jack changing.

![](images\image78.png)


Figure 71. status and score, respectively.

It looks like the Naughty / Nice status is control by a single value.
The score is maxed out and adding or subtracting a bit wouldn't change
much. However, if I can switch the Naughty / Nice status to naughty,
then the score will take on a more appropriate meaning. It *[must]{.ul}*
be the Naughty / Nice bit that needs to be changed.

A quick review of the naughty_nice.py code helps us confirm the
Naughty/Nice bit syntax and location of the bit in the block.

![](images\image79.png)


Figure 72. Code snippet from naughty_nice.py. As suspected, Naughty = 0,
Nice = 1.

![](images\image80.png)


Figure 73. The \"sign\" bit follows the score.

![](images\image81.png)


Figure 74. The "sign" and offset bits before change.

![](images\image82.png)


Figure 75. Bits after change.

Running naughty_nice.py again to confirm the change validates it was
successful.

![](images\image83.png)


Figure 76. Changed naughty/nice value to naughty.

Great! Now we need to confirm our MD5 collision worked and the MD5
hashes of Jack's altered block and our recreated version are the same. I
made some modifications to the script to perform an MD5 hash of the
block and print it out.

![](images\image84.png)


Figure 77. Side by side execution of naughty_nice.py with the original
blockchain.dat on the left and our modified blockchain on the right. MD5
hashes match!

Boom! It's a match!

Now we rehash the block with SHA256 and submit our answer.

![](images\image85.png)


Figure 78. SHA256 hash value of rebuilt block

It's good!

# Conclusion

After completing the blockchain objective, we head back to Santa's
office (bypassing the fingerprint scanner again) to give the elf the
good news. Once there we're thanked for solving the blockchain objective
and told to walk out to the balcony and be recognized. We find Santa,
Eve Snowshoes, and Jack Frost, decked out in an orange jumpsuit with a
rockin' Christmas jam playing. We're told Jack Frost had gifted Santa
the portrait and used it to control Santa Claus and sabotage his Naughty
and Nice list. Thanks to our efforts, we've saved Christmas!

  ------------------------------------------------------------------------------------------ ------------------------------------------------------------------------------
  ![](images\image86.png)
  ------------------------------------------------------------------------------------------ ------------------------------------------------------------------------------

*Figure* *79. Jack Frost before and after being captured. He doesn\'t
look so smug now.*

![](images\image88.png)


Figure 80. Group picture from Krinklecon3

# Lessons learned

As IT/Security professionals, we're constantly looking for new commands,
tools, and shortcuts to make our lives easier. Some of what I 'learned'
during this challenge I already knew but fell into the same traps. Other
things were completely brand new to me; things I may not ever directly
work with, but still provide a level of general knowledge and
understanding that I can apply to my day-to-day tasks.

### Documentation

> It's so easy to work through a challenge like this and not worry about
> documentation. My day to day requires tedious documentation and I'm
> always trying to better the detail I provide. During the challenge I
> found that my documentation was mostly on-point. I had what I needed
> to easily document my methodology to finding an answer -- except on
> ARP Shenanigans, which had a time limitation, and the last Blockchain
> challenge, where the material was completely new and initially a bit
> overwhelming to me. In those two instances I completely threw myself
> into the objective and did a poor job of documenting the final steps.
> In ARP Shenanigans, I had everything but the final file transfer,
> while in Blockchain Investigation II, I failed to grab some of my bit
> changes and the final SHA256 output. This required me to partially
> re-do, or in the case of ARP Shenanigans fully re-do, the objectives
> to fully document them for this report. I need continue working on
> maintaining thorough notes as part of my methodology.

### K.I.S.S aka "[K]{.ul}eep [I]{.ul}t [S]{.ul}imple [S]{.ul}tupid"

ALWAYS ALWAYS ALWAYS try the easiest methods first, and if it doesn't
work, double check your code. I spent far too much time on the
tag-generator attempting to fuzz the /image endpoint. I had tried
directory traversal to /etc/passwd early on, but it failed -- likely due
to not including the appropriate amount of directory changes. It wasn't
until much later, when I came across an OWASP document discussing
directory traversal[^16], that I decided to try again by cutting and
pasting their path to /etc/passwd. It worked and I kicked myself at
spending so much time chasing other methods when all I had to do was
double-check my initial traversal code.

One item that will help with this is creating or
"[borrowing](https://www.sans.org/blog/the-ultimate-list-of-sans-cheat-sheets/)"
a "cheat sheet" of web related OWASP attacks to cut and paste from on
future work.

### Asking for Help

I learned A LOT about blockchain and walk away with a basic
understanding of how it works and initially struggled wrapping my head
around the MD5 collision information required for the Naughty/Nice List
with Blockchain Investigation Part 2 objective. I read the slides and
listened to Ange Albertini's talk on YouTube [^17]to no avail. It wasn't
until I was \*nudged\* on the discord channel to make my hex editor look
like the editor in the UNICOLL slides, that it started to click for me.
Prior to that, I'd made it through the challenges without having to ask
for any help, but that's not idealistic in our industry. We need to stay
humble in our abilities and be okay asking for help from others. The
\*nudge\* I was given was small, but it was enough to put me on the
right track needed to understand and complete the entire objective.

And we pay it forward. Completing the objective gave me confidence and I
was able to share some \*nudges\* with others as well who were also
struggling.

![From zero to hero - Baby Win \| Meme
Generator](images\image89.jpeg)


# Comments & Suggestions

If you've ever had the opportunity to attend any sort of SANS event,
you're aware there's a survey for just about every activity. It's sort
of an inside joke among alumni and instructors, but it's completely
understandable as a method of continuous improvement in a technical
environment that is constantly evolving. It's in this spirit, that I
wanted to pass along my comments and suggestions.

1.  The music this year was super catchy and enjoyable. I especially
    liked "I'm the Real Santa" and "You're a Maen One, Mr. Grinch."
    Thank you for making it available outside of the challenge. I was
    able to share it with friends and family.

2.  The Discord channel was a great way to interact with others.
    Participants were professional and friendly.

3.  For the next Holiday Hack Challenge, evaluate providing a
    pause/screenshot opportunity when completing an objective. Some of
    the terminals closed out immediately upon completing the objective.
    For those of us who were documenting our progress with screenshots,
    that meant having to re-do the task to capture the final screen
    prior to completing, which at times became frustrating.

# Final Thoughts

I've fooled around with the SANS holiday hack challenge a few times in
the past, but this was the first year I set out to finish it. It was an
enjoyable and constructive use of my free time during the holiday
season. The challenges ranged from the simple to the complex and touched
on both offensive and defensive aspects of cyber security. Those just
starting out with the basics, as well as, those with years of experience
should each walk away with new knowledge.

Many thanks to \@edskoudis and team, the SANS Institute, and all the
sponsors for making this an annual holiday tradition of fun and
learning. I'm already looking forward to the next one.

[^1]: <https://www.gimp.org/>

[^2]: <https://sourceforge.net/projects/regshot/>

[^3]: <https://github.com/frgnca>

[^4]: <https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1123/T1123.md>

[^5]: <https://github.com/redcanaryco/atomic-red-team>

[^6]: <https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1074.001/src/Discovery.bat>

[^7]: <https://github.com/frgnca>

[^8]: <https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1123/T1123.md>

[^9]: <https://github.com/redcanaryco/atomic-red-team>

[^10]: <https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1074.001/src/Discovery.bat>

[^11]: <https://tools.ietf.org/html/rfc7465>

[^12]: <https://www.youtube.com/watch?v=RxVgEFt08kU>

[^13]: <https://gchq.github.io/CyberChef/>

[^14]: <https://github.com/kmyk/mersenne-twister-predictor>

[^15]: <https://speakerdeck.com/ange/colltris>

[^16]: <https://owasp.org/www-community/attacks/Path_Traversal>

[^17]: <https://www.youtube.com/watch?v=JXazRQ0APpI&t=227s>
