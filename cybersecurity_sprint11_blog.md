# 1.1 Overview
It was just another day in the office with us temps, college kids on break, readying ourselves for the monotony of data entry. The aroma of coffee wafted in the air with mumbles of "good morning." Then the door burst open and in flew the group's senior software developer, pale-faced and wild-eyed. He scanned the room and spotted a phone. He ran to it and began frantically dialing. We stared, unsure of whether to be amused or terrified. A moment later he said into the phone, "Shut up and listen. Go to my room. Grab all my printouts and burn them. You've got two minutes before the FBI shows up."

Apparently in his spare time, as our tinkering senior software developer was "just playing around," he somehow shifted the trajectory of a few random U.S. government satellites. In doing so, he piqued the interest of various government agencies, the brass of whom wanted to have a little sit-down, and not to talk about the weather.

# 1.2 About me
My name is Harris and though the tech field isn't new to me, doing a deep dive into cybersecurity is. And it's funny, because my first career mentor was a cybersecurity expert, and I've always had an interest in it as a field specialty due to the immense creativity it entails, (and you can even cue government satellites to do the Macarena, though I don't recommend it!), so I would've expected to have gravitated toward it sooner.

It took a recent full-circle moment to give me the final push to make this career change.

Anyway, this is my first cybersecurity blog post where I'll be describing what I experienced in the course of my bootcamp's Sprint 11 ("Deploy a SIEM"). I hope you get something out of it beyond just amusement at my mistakes. Though that's okay, too!

# 2. Modifications
# 2.1 Modification #1
For my first modification, I chose to add the ART Workstation as an agent to the SIEM. This is usually a straightforward process: You tell the SIEM you want to deploy a new agent (from the Wazuh main menu, `Agent management` → `Summary` → and then click the `Deploy new agent` link`).

It asks a few questions, such as:
- Which operating system is running on the new agent device
- What's the SIEM server's address
- What name do you want to give your new agent, and
- What group(s) should the new agent belong to (groups help centralize administration over multiple agents simultaneously)

Then the SIEM generously displays the code that needs to be run on the new agent device to officially welcome it into the family. By hovering over the code, a 'Copy' link appears, and when you click it, the code is copied to your clipboard. Then all you have to do is simply paste the clipboard (which will contain the code it copied) on the new agent into a terminal window for UNIX systems, or into a Powershell window for Windows systems, and then you execute it, i.e., press the `ENTER` key. Lastly, you start the service (check your documentation, but usually you type in `sudo systemctl start wazuh-agent` on UNIX systems or `net start WazuhSvc` on Windows systems). And viola! Or in my case, viol-*nah*.

Norton Anti-Virus, the David Lee Roth of AV software packages, decided it was being deprived of attention, so it jumped into the spotlight on the step where you copy the code from the SIEM onto your clipboard. I hovered over the 'Copy' link, but when I clicked it, Norton popped up and claimed the code I was about to copy was Satanically evil. Now, if it isn't obvious, let me state that I'm not a big fan of Norton, but every time I try to quit using them, they offer a deep discount. It's like The Godfather making an offer you can't refuse. Just when you try to get out, they pull you back in! In their defense, the code was evil, technically speaking, but Norton should've given me the choice to allow an exception, kinda like, "Leave the gun. Take the cannoli." As a Norton user, I should be empowered to decide when I want to take a gun or a cannoli, figuratively speaking, of course.

I could've disabled Norton and copied the code, but Norton would have had VERY strong words for me. And my PC would be exposed for that brief moment. Inspecting the page to find the code snippet proved too difficult, so I decided to manually copy the code by typing it out. I pulled up Notepad and began typing one character at a time. I then copied the code from Notepad and pasted it into the ART Workstation, ran it, and fired the service up.

And like magic, the ART Workstation showed up in the SIEM! But I noticed it was under a different name (`student-host-windows.megaquagga.local`). It was also missing the group information.

I realized it got the name for the ART Workstation from DNS. And that, coupled with the missing group information made me look back at the command (the code) I had entered into the ART Workstation Powershell window. A chunk of the necessary code was missing. When I manually copied the code, I didn't realize it wrapped around to the next line, and the two arguments that specified the name I wanted to give the agent, along with the groups in which I wanted it to belong were on the next line (that I neglected to copy). In other words, there were no typos in the code I executed - It was just missing some of the arguments, such as the agent name and the groups to which it should be added, both of which were filled in with defaults - the DNS name of the workstation and no it wasn't added to any groups because none were specified.

So, no biggie, I figured. I'd just go into the SIEM, rename the agent, or worst case, delete the agent and re-add it. But NoooooOOO! There is no feature for renaming agents in the Wazuh SIEM. There isn't even a delete feature! You can either run API code (with a prerequisite of a masters degree from NASA) or you can run through hoops to delete them and add them back in.

Per the Wazuh documentation, I had to find my way into the Wazuh server and run a command-line interface to delete the agent. That worked and the agent no longer appeared in the SIEM.

But then it reappeared, using a different agent ID! It was like trying to kill an agent in the movie The Matrix!

I figured I'd have to delete it from the ART Workstation first, but none of the command-line interface commands worked. Finally, it occurred to me that I was working on a Windows system, so why not delete it like any other software on a Windows system by using the 'Add or remove programs' GUI? I did that, it worked like a charm, I deleted the agent from the SIEM server (again), and I added it back in, but this time, I defined all the proper parameters. And then everything was peachy.

# 2.2 Modification #2
I opted to do a second modification, which was to establish the ‘observer’ Apache logs as a new log source in the SIEM.

To do so, I added the following to (`/var/ossec/etc/ossec.conf`) the ossec.conf file:
>
`<localfile>`
&nbsp;&nbsp;&nbsp;&nbsp;`<log_format>apache</log_format>`
&nbsp;&nbsp;&nbsp;&nbsp;`<location>/var/log/apache2/access.log</location>`
`</localfile>`
<br>`<localfile>`
&nbsp;&nbsp;&nbsp;&nbsp;`<log_format>apache</log_format>`
&nbsp;&nbsp;&nbsp;&nbsp;`<location>/var/log/apache2/error.log</location>`
`</localfile>`

Next, I restarted the agent with: `sudo systemctl restart wazuh-agent`

And then I brought up the home page of the website into the browser on the Blue-Team's workstation: `https://10.170.0.20/`

Lastly, I verified that the access.log registered my home page request (first by checking that the file date/time was updated, then by displaying the last few lines of the file):
> `ls -l /var/log/apache2/access.log`
> `tail /var/log/apache2/access.log`

# 3. Experiments

# 3.1 Experiment #1

For my first experiment, I ran Atomic Red Team test T1059.003 (“Command and Scripting Interpreter: Windows Command Shell”) from the ART workstation against the target device, ad01.

For the sake of convenience, I created a variable called `$sess` (borrowed from the coursework) and set it to the following string: `New-PSSession -ComputerName 10.170.0.10 -Credential administrator`

This defined parameters for how the attack would deploy on the target device (‘ad01’), including the method of execution (Powershell), the IP address of the target device, and the ID under which the attack would run (‘administrator’).

To determine which, if any, prerequisites existed (required files) for the attack, while in a Powershell on the ART Workstation I ran: `Invoke-AtomicTest T1059.003 -Session $sess -GetPrereqs`

Out of curiosity, I decided to run the attack locally first, which I did with the following: `Invoke-AtomicTest T1059.003`

And then hilarity ensued.

A barrage of WordPad windows began opening on my screen in quick succession. I had to frantically fight to bring the original Powershell window back into focus, and finally, my multiple CTRL-C taps on the keyboard stopped the madness.

The business of running exploits is new to me, and what had happened was, for some unknown reason, I mistook the ".003" portion of the attack name ("T1059.003") to mean Test #3. Clearly I was wrong. For cleanup, I had to close each window (I don't think a `-cleanup` would've worked because each WordPad window had an error popup, complaining they couldn't find a file they attempted to read - See image 1 below).

Image 1![[wordpad_error.png]]
What I should have run was: `Invoke-AtomicTest T1059.003 -TestNumbers 3`

That would've specified which (sub)test I had intended to run.

After looking into which attack actually ran, it was Test #4 (Apparently the attack was coded in a way where Test #4 was the default test to run if no -TestNumbers were specified), and while I feared the system may have run out of resources while all the WordPad windows were popping up, I learned that at most, there would've been a maximum of 75 wordpad windows, which is the default maximum for that test.

Following that mishap, I was ready to run the test on the target system, but I couldn't help but wonder how the same attack I had run locally, where I neglected to specify test #3, would affect the target system. So, I ran the same command that spawned a WordPad bomb on the local system on the target system as follows: Invoke-AtomicTest T1059.003 -Session $sess

Interestingly, no WordPad windows opened on the target system, and the likely cause was that the attack was done within a noninteractive session, meaning there was no GUI and no way to create a window handle. Therefore, rather than open up to 75 WordPad windows, the exploit just produced an error. And as it turned out, the SIEM picked up the attempt to launch wordpad.exe:

> `data.win.eventdata.parentCommandLine` contained:  
>   
> `\"C:\\Windows\\system32\\cmd.exe\" /c \"for /l %x in (1,1,75) do start wordpad.exe /p C:\\Users\\ADMINI~1\\AppData\\Local\\Temp\\AtomicRedTeam\\..\\ExternalPayloads\\T1059_003note.txt\"`

Pretty neat!

# 3.2 Experiment #2

For my second experiment, I ran Atomic Red Team test T1027 (“Execute base64-encoded PowerShell -Test #2 - Execute base64-encoded PowerShell“) from the ART workstation against the same target device, ‘ad01’.

I reused the `$sess` variable (which was set in Experiment #1). No prerequisites were required, and I ran the test with the following via Powershell: `Invoke-AtomicTest T1027 -TestNumbers 2 -Session $sess`

The SIEM picked up instances of attempts to execute an encoded command with the field `data.win.eventdata.parentCommandLine` showing:

> `\"powershell.exe\" &amp; {$OriginalCommand = 'Write-Host \\\"\"Hey, Atomic!\\\"\"' $Bytes = [System.Text.Encoding]::Unicode.GetBytes($OriginalCommand) $EncodedCommand =[Convert]::ToBase64String($Bytes) $EncodedCommand powershell.exe -EncodedCommand $EncodedCommand}`

# 3.3 Experiment #3

For my third experiment, I ran Atomic Red Team test T1078.001 (“Valid Accounts: Default Accounts - Test #2 - Activate Guest Account”) from the ART workstation again, against the target device, ‘ad01’.

I reused the `$sess` variable (which was set in Experiment #1). No prerequisites were required, and I ran the test with the following via Powershell: `Invoke-AtomicTest T1078.001 -TestNumbers 2 -Session $sess`

The SIEM picked up an instance of an attempt to activate the Guest account:

`data.win.eventdata.parentCommandLine` contained:

> `\"cmd.exe\" /c net user guest /active:yes`

Very cool that I was able to impersonate a different ID through this test!
# Conclusion
# 4.1 Summary of experimental findings
I found this Sprint to be well written and incredibly interesting! I conducted three Atomic Red Team tests, and in the first test, (T1059.003), I accidentally launched a "WordPad bomb" on the local system due to my confusing the ".003" of the test name with what should have been a parameter of `TestNumbers 3`. Running the same test remotely resulted in no visible windows due to it being a non-interactive session, but the SIEM successfully logged the attempted exploit. The second test, (T1027), executed a base-64 encoded PowerShell command remotely, which was also detected by the SIEM. And the last test, (T1078.001), attempted to activate the Guest account remotely, with the SIEM logging the attempt with the corresponding `net user` command. Very cool to be able to impersonate another user on a remote system, which, in theory, could act as a launchpad for attempting to upgrade privileges.

# 4.2 Advice on avoiding mistakes
A simple typo, or accidentally omitting parameters when creating a new agent for the Wazuh SIEM, can lead to a lot of headaches, requiring lots of research, time, and energy to fix something caused by carelessness. So, it's worth paying close attention to detail when executing code that affects multiple systems, especially when dealing with a system that may not be the most user-friendly for certain operations (in this case, the Wazuh SIEM when it comes to renaming or deleting an agent).

Also, again, it's worth paying close attention to detail when running exploits, even if they're just simulations. They're still attacks, and they could do damage. The mistake I made in confusing '.003' with 'TestNumbers 3' could've potentially choked the system out of resources, even with the test having a limit of 75 windows with its "WordPad bomb."

# Final thoughts
# 5.1 The coolest thing I learned
The Atomic tests were mentioned earlier in this bootcamp, but seeing them in action was amazing! I would say those tests are the coolest thing I learned in this Sprint, if not the entire course so far (the only exception being reverse shells, which floored me). And the Atomic tests being open source, where the good people of the Internet keep them current, boosts my faith in humanity.

# 5.2 One piece of advice
When you take a bootcamp, a lot of the course material is compressed, and therefore, the course pacing tends to be faster than a normal course or perhaps an instructional book. If you couple that with trying to get through the bootcamp quickly, you may find that you're not paying attention to details at the level you should. You may be taking notes without fully digesting them, e.g., a link to a website containing material that expands on a topic may be briefly mentioned, and you may skip over it out of a desire to keep pushing forward. However, in doing so, you may be denying yourself valuable information.

The key is finding a healthy balance between what you should skip, skim, and fully dive into. The good thing (as I learned within this Sprint) is that the material in this bootcamp will be available for life, but it's important to grasp as much as you can as a service to yourself, and ultimately your employer, as you make your way through it.

# 5.3 My favorite resource
My favorite resource has to be Gemini AI. I had been using other AI platforms, but when I started taking this bootcamp, I figured Google may be better for finding and explaining more technically-oriented details vs. other AI platforms. I take a trust-but-verify approach with AI, and so much of the value that comes out of it is determined by the questions and context that go in. It shouldn't be used as a crutch because, at least as of now, we as humans bring so much more to the table. Therefore, it should be used as a helper agent, performing relatively simple tasks such as finding answers across the web more efficiently than we can with Google or other search engines. It's also great with specifying or correcting syntax, rewording, etc.. It's a powerful tool, but remember, garbage in, garbage out. It's worth verifying the answers it gives.

# 5.4 Thank you (gratitudes)!
I recognize that we as students are probably expected to thank those who authored software or articles that aided the cybersecurity community, but I'd like to thank those who put me on this path. For me, it just feels like a more authentic response.
# 5.4.1 Mr. Schneider
When I was barely a teenager, I was fortunate to be "friends" with Mr. Schneider, an electronics engineer who lived a couple houses away from my childhood home. He was soft-spoken and kindhearted, and his door was always open to me. I would ring the bell and his wife always warmly welcome me in, directing me toward the basement.

In Mr. Schneider's dungeon, or lab, amidst the permanent fog of apple-scented pipe smoke were devices ordinarily seen in sci-fi movies, along with a sea of wires, transistors, resistors, capacitors, diodes, breadboards, and on and on.

I showed him my latest computer work and he taught me the fundamentals of electronics. I would bring home bizarre gadgets that regularly flummoxed my family, like the buzzer made from a carved-up coffee can, a wire coil and a 9-volt battery, or the light bulb that came to life when I held its probes in separate hands while rubbing my socks on the carpet.

I aced Radio Shack's electronics course and thought I too, would become an inventor, like Mr. Schneider. But I went off to college to do all that zany college kid stuff. One area I focused on was computer science, and I can credit Mr. Schneider, my first mentor, for helping put me on that path, which ultimately became my career.

I came home from college on a break and learned Mr. Schneider was battling lung cancer. I asked him if we could get together again in his basement, and through labored breath aided by his remaining lung, he said, "When I'm feeling better." I don't know if denial prompted my question, or whether it was my way of letting him know he'll always be valued, whether down in his basement or forever in my memory.

# 5.4.2 Craig
I had graduated college just as the economy went into a recession. I was facing the typical Catch-22, where employers only wanted to hire those with experience and those without experience couldn't get hired, only made worse by the recession. So, I took a temp job, filling in for a secretary who was going on a two-week vacation.

The company was UNIX System Laboratories, and the secretary showed me their standard text editor, 'vi'. And this wasn't the 'vi' of today, where keyboard cursor keys move you around the screen. As in the movie "Airplane!" when it was suggested to the captain that they turn on the searchlights to aid the troubled aircraft, his response was, "No. That's just what they'll be expecting us to do!" And in all fairness, 'vi' came to be at a time when most keyboards didn't have cursor keys, possibly because they were all eaten by dinosaurs.

But I digress. The secretary told me I wouldn't be able to learn 'vi' in the time she'd be away, but to do my best with it and the other clerical tasks.

It turned out I really took to 'vi' and by the time the secretary returned from vacation, I began training her on how to use it. I was also building various utilities for staff, one of which was a conference room reservation system (before such software was readily available).

The company was full of geeks who were more than capable of hacking my system, stealing rooms away from others for themselves, so I needed a secure software design, but I had no idea where to begin.

I asked around for someone who could help, but I was considered bottom-rung material. People were friendly but no one had time to hold the hand of a bright-eyed, bushy-tailed, fresh-out-of-college temp. And the learning curve was too steep for me to take on by myself.

Finally, I met Craig, one of the brainiacs from the UNIX Security group, and he was willing to help. He and I met regularly and he taught me fundamentals of UNIX security. His lessons were invaluable, but the most important thing I learned was that being kind and patient doesn't cost anything. And years later in my career, I tried to follow Craig's example in how I mentored others.

In a full-circle moment, I recently met up with Craig at a UNIX System Laboratories reunion. I was scanning the room, looking for people I might recognize when someone tapped my shoulder and said my name. I immediately recognizing him, despite all the time gone by.

I had been hoping he'd attend, because as I'd told friends and family, I wanted to thank him. And I began to do just that when, to my surprise, I became extremely emotional and got completely choked up. I don't think I could've made the moment any more awkward as I silently fought back tears with Craig standing in front of me, probably wondering what on earth was going on. I finally got the words out, letting him know how pivotal he was at the start of my career, and how his kindness and patience stayed with me, which I attempted to pass onto others.

It was nice to be able to express my gratitude to Craig after all these years. He's made his way up the ladder into a big position at a big company, all so well deserved. We now keep in touch, and he recently sent me an article that highlighted the urgent need and growing gap for cybersecurity experts in the IT field. It wasn't a push or even a nudge, but then, with Craig, it never was. Yet it was all I needed to decide to join Craig in making the world a safer place. And with that, I tossed on a white hat and started my deep dive into cybersecurity.

# References

**[Adding a new endpoint to the SIEM: Windows](https://tripleten.com/trainer/csa/lesson/3463381e-9e78-4223-87a4-a56b8a933d4f/)** by TripleTen (Cybersecurity bootcamp):
Clear and well written instructions on how to add a new Windows-based endpoint to a Wazuh SIEM.

**[Gemini AI](https://gemini.google.com/)** by Google (Google's free AI platform):
Ask and ye shall receive. And as with so many AI platforms, ye should trust but verify.

**[Configuration for monitoring log files](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/monitoring-log-files.html)** by Wazuh (product documentation):
Guide to configuring the Wazuh agent `ossec.conf` file to collect logs from specific log files on a monitored endpoint.

**[Configuration for monitoring log files: log_format](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html#log-format)** by Wazuh (product documentation):
Guide to modifying Wazuh endpoint configuration files.

**[Atomic Red Team: T1059.003 - Command and Scripting Interpreter: Windows Command Shell](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1059.003/T1059.003.md)** by Atomic Red Team `[2025-02-13]`:
Describes various simulated attacks with payloads executing via the Windows command shell (`cmd`).

**[Atomic Test #2 - Execute base64-encoded PowerShell](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1027/T1027.md#atomic-test-2---execute-base64-encoded-powershell)** by Atomic Red Team `[2025-02-13]`:
Describes a simulated attack that creates base64-encoded PowerShell code and executes it.

**[Atomic Test #2 - Activate Guest Account](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1078.001/T1078.001.md#atomic-test-2---activate-guest-account)** by Atomic Red Team `[2025-05-01]`:
Describes a simulated attack that actives the default Guest account on a Windows system.
