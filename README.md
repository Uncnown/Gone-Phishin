# Week10 Lab Exercise - Gone Phishin'

![Image](http://i.imgur.com/dFoYLJH.png)

In this lab, we'll take a hands-on look at phishing, which can involve a number
of different approaches, and a growing set of tools is available to the
aspiring social engineer to assist with this. It's important to remember,
though, that phishing is a subset of SE in general, and that many SE techniques
such as [pretexting](http://searchcio.techtarget.com/definition/pretexting) or
[waterholing](http://searchsecurity.techtarget.com/definition/watering-hole-attack)
are beyond the scope of what we can usefully
simulate or even demonstrate, either because they are highly dependent upon
knowledge about the target or because they involve manipulating a human
directly (or both). Because the targets and systems involved in SE are usually
in a constant state of change, what works in one situation will often not work
in others, or work reliably at all, and SE often involves at least a certain
amount of luck. Many of the effective and high-value examples of SE are unique
to their circumstances and thus hard to generalize.

And while we can cover some of the tools and techniques, one additional problem
will become clear over the course of the lab: this stuff doesn't always work,
and the well-understood variations typically won't work against modern
infrastructure. In previous weeks we've sidestepped this by attacking
intentionally vulnerable targets; hacking the latest version of WordPress is a
lot harder than hacking one that's three years old. In this lab, we'll come up
against modern, updated infrastructure and see first hand how some of these
techniques fall short. But even that is instructive: security is many ways an
arms race, and today's tools and tech will often not work tomorrow. Part of
being a successful security researcher or hacker is adapting to this state of
affairs. We take the time to understand these known, sometimes obsolete
techniques so we can move closer to the frontier, where the war is being
fought.

## Topics in Focus

* [Social
  Engineering](http://guides.codepath.com/websecurity/Social-Engineering)

### Milestone 0: Setup

Spear Phishing refers to a targeted phishing attack against one or more
specific individuals, leveraging known information about them to manipulate
them into performing some specific action. To demonstrate this, we'll look at
another tool which runs in Kali: [Social Engineering
Toolkit](https://github.com/trustedsec/social-engineer-toolkit) (SET).

For this lab, we'll need to create a new Kali docker container with slightly
different settings. Recall that our current kali container was created using
docker's host networking mode, which allows the container to inherit the host's
networking configuration. For this lab, we'll need a container which also
exposes a port to the host in a method similar to the WordPress container.
We'll be cloning the kali container, so it will be preserved, and the whole
process shouldn't take much longer than 10-15 minutes.

On your host machine, stop the `kali` container, clone it, and run a new
instance named `kalise` with mapped ports:

```
# stop existing kali container, if running
$ docker stop kali
# clone the kali container image to a new image named 'kalise'
# this will take 2-3 minutes
$ docker commit kali kalise
# run the cloned container mapping container port 80 to host port 666
$ docker run -p 666:80 -itd --name kalise kalise
$ docker exec -it kalise bash
# update apt
root@kalise:/# apt-get update
# install set and rar via apt
root@kalise:/# apt-get install -y set rar
# run ifconfig to determine the new container's IP address
root@kalise:/# ifconfig
```

By now you should be able to determine the IP of the container from the output.
Note the networking configuration is much simpler since we're not running in
host mode. You'll need this for the following milestones.

## Milestone 1: Hello, SET

It's not technically very difficult to get started with phishing: anybody can
send out a mass mail with a malicious link or even a payload, and even though
the leading email providers like Gmail have filters in place that have made
this much harder than it used to be, if one is clever enough and casts a wide
enough net, a few users still might get caught in it. Still, we've come a long
way, and gone are the days when you could [hack millions just by sending them a
love note.](https://en.wikipedia.org/wiki/ILOVEYOU)

For this exercise, you'll need to create a throwaway
[Gmail](https://accounts.google.com/SignUp?hl=en) account to use with SET.
Don't use your regular gmail account if you already have one.

![Image](http://i.imgur.com/7qx09hH.png)

In your new `kalise` container, run `setoolkit` and accept the terms and
conditions. Like Metasploit, SET is a command-line tool that creates its own
shell with specific commands available. You'll see a root menu and the `set>`
command prompt.
```
 Select from the menu:

   1) Social-Engineering Attacks
   2) Penetration Testing (Fast-Track)
   3) Third Party Modules
   4) Update the Social-Engineer Toolkit
   5) Update SET configuration
   6) Help, Credits, and About

  99) Exit the Social-Engineer Toolkit

set>
```

We'll start with a very simple mailer walkthrough. The menus in SET are nested,
so you generally will need multiple options to drill down to the specific
exploit/tool using the option numbers presented. Enter the option numbers for
the labels below:

Choose `Social-Engineering Attacks`
Choose `Mass Mailer Attack`
Choose `E-Mail Attack Single Email Address`

At this point, SET will prompt you for a series of values. For this example,
you'll send a basic test email with a harmless payload (one of the Kali desktop
images) both from and to your test account (displayed as
`herephishyphishyphish@gmail.com` in the snippet below). Use the following
snippet as a guide, but enter anything you like for "FROM NAME", "subject", and
the message body.

```
set:phishing> Send email to:herephishyphishyphish@gmail.com

  1. Use a gmail Account for your email attack.
  2. Use your own server or open relay

set:phishing>1
set:phishing> Your gmail email address:herephishyphishyphish@gmail.com
set:phishing> The FROM NAME the user will see:Trusted Friend
Email password:
set:phishing> Flag this message/s as high priority? [yes|no]:no
Do you want to attach a file - [y/n]: y
Enter the path to the file you want to attach: /usr/share/desktop-base/active-theme/grub/grub-4x3.png
set:phishing> Email subject:Something Special 4 U
set:phishing> Send the message as html or plain? 'h' or 'p' [p]:p
[!] IMPORTANT: When finished, type END (all capital) then hit {return} on a new line.
set:phishing> Enter the body of the message, type END (capitals) when finished:Shall I compare thee to a summer's day?
Next line of the body: END
[*] SET has finished sending the emails

      Press <return> to continue
```
If all goes well, you should have a message in your gmail inbox:

![Image](http://i.imgur.com/uqPj5sl.png)

Of course, normally, the "to" and "from" emails would be different, and the
payload wouldn't be harmless.

## Milestone 2: Try a Real Payload

Let's try one of the real payloads with SET. After completing the above, you should see the SET menu redisplayed.

Choose Spear-Phishing Attack Vectors
Choose Create a FileFormat Payload
You'll see a long list of payload options:

 Select the file format exploit you want.
 The default is the PDF embedded EXE.

           ********** PAYLOADS **********

   1) SET Custom Written DLL Hijacking Attack Vector (RAR, ZIP)
   2) SET Custom Written Document UNC LM SMB Capture Attack
   3) MS15-100 Microsoft Windows Media Center MCL Vulnerability
   4) MS14-017 Microsoft Word RTF Object Confusion (2014-04-01)
   5) Microsoft Windows CreateSizedDIBSECTION Stack Buffer Overflow
   6) Microsoft Word RTF pFragments Stack Buffer Overflow (MS10-087)
   7) Adobe Flash Player "Button" Remote Code Execution
   8) Adobe CoolType SING Table "uniqueName" Overflow
   9) Adobe Flash Player "newfunction" Invalid Pointer Use
  10) Adobe Collab.collectEmailInfo Buffer Overflow
  11) Adobe Collab.getIcon Buffer Overflow
  12) Adobe JBIG2Decode Memory Corruption Exploit
  13) Adobe PDF Embedded EXE Social Engineering
  14) Adobe util.printf() Buffer Overflow
  15) Custom EXE to VBA (sent via RAR) (RAR required)
  16) Adobe U3D CLODProgressiveMeshDeclaration Array Overrun
  17) Adobe PDF Embedded EXE Social Engineering (NOJS)
  18) Foxit PDF Reader v4.1.1 Title Stack Buffer Overflow
  19) Apple QuickTime PICT PnSize Buffer Overflow
  20) Nuance PDF Reader v6.0 Launch Stack Buffer Overflow
  21) Adobe Reader u3D Memory Corruption Vulnerability
  22) MSCOMCTL ActiveX Buffer Overflow (ms12-027)

set:payloads>
Take a moment to read the available options. There's a clear pattern here of malicious payloads embedded in the kinds of files you'd typically see as email attachments: PDFs, ZIP, even RTF.

Which two software companies are heavily represented on this list?
Which operating system would most of these exploits require?
That last one is especially noteworthy, because if the target isn't using that OS, most of these will not work. Something to consider...but for the purposes of this lab, it doesn't matter if you don't have the right OS for this.

Let's proceed with option 6) Microsoft Word RTF pFragments Stack Buffer Overflow (MS10-087)

This payload takes advantage of an issue in the Microsoft Word RTF parser, which affects all versions of Microsoft Office prior to the release of the MS10-087 security patch. So this payload would not only be limited to users of a specific OS but who are also using an outdated version of MS Office.

As with so many of these payloads, the specific issue being exploited is a buffer overflow. It's not necessary to understand the details, but this is an extremely common type of vulnerability for apps written in C and C++. If you're game, you can read more about how it works in this particular case here.
After choosing this option for the payload, you'll be presented with a further list of options for how the payload, once delivered, will work:

   1) Windows Reverse TCP Shell              Spawn a command shell on victim and send back to attacker
   2) Windows Meterpreter Reverse_TCP        Spawn a meterpreter shell on victim and send back to attacker
   3) Windows Reverse VNC DLL                Spawn a VNC server on victim and send back to attacker
   4) Windows Reverse TCP Shell (x64)        Windows X64 Command Shell, Reverse TCP Inline
   5) Windows Meterpreter Reverse_TCP (X64)  Connect back to the attacker (Windows x64), Meterpreter
   6) Windows Shell Bind_TCP (X64)           Execute payload and create an accepting port on remote system
   7) Windows Meterpreter Reverse HTTPS      Tunnel communication over HTTP using SSL and use Meterpreter
Most of these are reverse shells, which you should be familiar with from working with Meterpreter. In fact, you can see there are a couple of Meterpreter options.

Let's be optimistic and pick Windows Meterpreter Reverse_TCP (X64), which would give us extensive control over the target system if successful.

After picking this option, SET will ask you to enter the IP address for the payload listener (LHOST), which should be the IP of the server Meterpreter will try to connect to if the payload is delivered and activated successfully. You can just use 127.0.0.1 for now and use the default (443) for the next prompt about the the port since we're not going to try to activate the payload.

The next prompt will ask you to either keep the default filename (moo.pdf) or to rename the file. The name of the file as it appears in the email is important, so you'd want to pick something that will tempt the user to open it. And you'd want to use the .rtf extension so it opens in MS Word.

For the next option, choose 1. E-Mail Attack Single Email Address to send to just a single address.

The following option will ask you if you want to use a pre-defined template email or a custom one. A spear-fishing attack would likely use a custom email, in which you specify the subject and body as before. But the pre-defined options are amusing:

   Do you want to use a predefined template or craft
   a one time email template.

   1. Pre-Defined Template
   2. One-Time Use Email Template

set:phishing>1
[-] Available templates:
1: Status Report
2: Have you seen this?
3: Computer Issue
4: Strange internet usage from your computer
5: Dan Brown's Angels & Demons
6: Order Confirmation
7: WOAAAA!!!!!!!!!! This is crazy...
8: New Update
9: Baby Pics
10: How long has it been?
set:phishing>
The following prompts should be familiar; use the same values as before:

set:phishing> Your gmail email address:herephishyphishyphish@gmail.com
set:phishing> The FROM NAME user will see:Someone Special
Email password:
set:phishing> Flag this message/s as high priority? [yes|no]:no
set:phishing> Does your server support TLS? [yes|no]:no
You should get an error at this point.

[!] Unable to deliver email. Printing exceptions message below, this is most likely due to an illegal attachment. If using GMAIL they inspect PDFs and is most likely getting caught.
Press {return} to view error message.
(552, '5.7.0 This message was blocked because its content presents a potential\n5.7.0 security issue. Please visit\n5.7.0  https://support.google.com/mail/?p=BlockedMessage to review our\n5.7.0 message content and attachment content guidelines. p16sm12601292pgc.4 - gsmtp')
[*] SET has finished delivering the emails
Alas, Gmail is wise to our script kiddie ways. As users, this is reassuring...Google is scanning the attachment for a known malicious signature and preventing us from sending the mail (even to ourselves). As hackers, this is frustrating, but illustrative of the challenges facing the would-be spear-phisher these days. For sending anything more than a link to a malicious site, it's not nearly as easy as it used to be, but that's partly because it's so effective when it does work.

So how would we get this payload through? Nominally, we'd have to find a mail relay that didn't have this attachment content scan implemented. Most major providers (hotmail, yahoo, etc.) have some measures like this in place, but not all, and not all of them are as sophisticated. But then even if we were able to send this mail through another relay, if our target is a gmail address, the same scan would likely be applied to the mail on the incoming side and it would just be bounced.

To get this into a gmail inbox, the payload would probably need to be customized in some way to evade signature-based detection, and maybe even re-engineered to avoid well-known exploits like this one. It would also probably need to come from an address associated with the target to avoid being flagged as spam (after all, how often do you get RTF attachments from strangers?). We'll leave it as a very optional bonus for you to find a way to get this particular payload delivered using SET (Gmail is pretty solid, so a softer target would help), but for now, since many phishing attempts don't include any payload other than a malicious link, let's look at what might be on the other end of such a link.

If you've ever tried to manage your own (legitimate) mass mailing app, you're probably familiar with some of these issues, since in recent years, major providers like Gmail have started more aggressively fighting spam by restricting how emails from "untrusted" sources are handled. These days, even for sending legitimate mass mailings, many people have to use a third-party like MailChimp or SendGrid to avoid getting caught in filters.
Milestone 3: Fakebook
Picture this: you're sitting with your favorite laptop in your favorite cafe, using their public Wi-Fi and browsing Facebook. The Wi-Fi connection drops suddenly, and you have no internet connection. You open your wireless network settings and don't see the cafe's network listed, so you're just about to get up and ask the overworked barista about it when you see the network SSID pop up again. You reconnect and head back to Facebook, but you've been logged out, so you log back in and continue browsing. None of this is particularly unusual. It's a busy morning in the cafe, and lots of people are using the cafe's single wireless AP, so a spotty connection makes sense given the load. It's also not too surprising that you were logged out of Facebook as the result of a connection drop, and anyway, your login worked and nothing seems amiss now.

You probably didn't notice the quiet person sitting in the corner with a laptop who just ran an evil twin attack on the entire cafe. You'd have no way of knowing her backpack contained a USB wireless interface capable of packet injection, enabling her to kick everyone off the cafe's Wi-Fi and then spoof it using the same SSID so that everyone would reconnect to her rogue AP instead. And you probably didn't have more than a few seconds to notice that the URL of the Facebook login page to which you were redirected wasn't facebook.com but was an IP address (pointing to her laptop) or that it was missing the little green lock symbol your browser displays for secured URLs.

The reason you probably didn't notice that last part was because the login page itself looked exactly like the real Facebook login page, down to the fonts and images and even the favicon, and after typing in your username and password, you were seamlessly redirected back to the real wireless AP and the real Facebook (to which you were already authenticated anyway).

This isn't even a very sophisticated attack chain with the right equipment, but still more than we can reliably emulate without it. However, we can use SET to make one of these fake login pages pretty easily. From the root SET menu:

Choose 1) Social-Engineering Attacks
Choose 2) Website Attack Vectors
Choose 3) Credential Harvester Attack Method
Choose 2) Site Cloner
SET will ask you for the IP address for the POST; this is the IP that the fake login page will post to, so use the IP of the kalise container you determined in milestone 0.

Next, SET will ask you for the site to clone, so enter http://www.facebook.com and then SET will run a server with the cloned site:

set:webattack> Enter the url to clone:http://facebook.com

[*] Cloning the website: https://login.facebook.com/login.php
[*] This could take a little bit...

The best way to use this attack is if username and password form
fields are available. Regardless, this captures all POSTs on a website.
[*] The Social-Engineer Toolkit Credential Harvester Attack
[*] Credential Harvester is running on port 80
[*] Information will be displayed to you as it arrives below:
You should be able to view this from your host at http://localhost:666:

fakebook

A few things to note:

It's not perfect, but it's close enough to fool most users
It's fast enough that a hacker could potentially generate it on-the-fly (useful for harvesting credentials from more obscure sites)
SET is listening for the POST this page would create, ready to intercept all the credentials and write them to the filesystem, but...
Because we're running this inside a docker container, the POST will try to go to the IP of the Kali container instead of the localhost-mapped port (localhost:666) and thus will fail. Normally, running Kali as the host OS, this wouldn't be an issue, but in this case you'll have to do some more setup to get the intercept to work.

Milestone 4: Steal Your Face(book)
Challenge: Make the fake login POST work on your host machine such that SET is able to capture at least the POST data

SET is listening on port 80 inside the kalise container, but we don't have a way to access the Kali container by its IP due to the limitations of Docker. But you have all you need to make this work such that the POST gets redirected via localhost:666 back to the kalise container. Hints:

Use burp as a proxy, setup in the same way you did for earlier labs
Manual approach: capture the POST request, send it to the Repeater and replay it against localhost:666
Bonus points: Use Proxy > Options to configure burp to automatically handle the redirect
Monitor the SET console for information about what's being captured
Hit CTRL-C when you think you've got it. Exit SET and look for the captured XML data in ~/.set/reports
What is the AJAX on the login page doing?
Did SET capture the credentials? Why or why not?
Facebook's isn't your average login page, so this will take some time to understand. If you aren't able to capture the credentials, it's more important to understand why than to actually make this work with SET. One thing to remember is that because SET is cloning the page, you'd have options to make this easier on yourself, since it's more important that the page look like the real thing than work like the real thing.

We're also making it a bit harder on ourselves than it has to be with the Docker setup, but time with burp is time well spent. It's fun to play with all the Kali tools, but people who do this kind of work with regularity will tell you that burp is one the top five tools they use all the time.

Milestone 5: SE In Situ
The most creative uses of social engineering often target specific groups or individuals, taking advantage of more than just lax security on the part of the end user, but also weak infosec or procedures on the part of large organizations and companies. As such, it can be difficult to simulate SE techniques in the same way we've simulated WordPress hacks. Instead of a static target with predictable vulnerabilities, SE targets are people with mostly predictable behavior: an end user who's reusing passwords across sites, a help desk technician following a specific playbook.

Even though we can't simulate situations like this, it's helpful to look at some examples. Read/watch the following:

How Apple and Amazon Security Flaws Led to My Epic Hacking
 What Happens When You Dare Expert Hackers To Hack You
In the cases, we see a user compromised through techniques that exploit security measures partially beyond his or her control. Challenge questions:

What vulnerabilities were beyond the control of the user?
What if anything could have been done by the user to mitigate the severity of the attack?
Now, think back to the fake Facebook login scenario above. Assuming a successful compromise, in which the user's Facebook username and password were successfully intercepted, answer the following:

What could the user do to mitigate this, making a successful login impossible for the attacker even with the credentials? (Hint: FB offers this as an option; not all sites do)
Why might the username/password still be of value to the attacker even if she can't use them to login to Facebook? (Hint: think about how users come up with passwords)
Bonus Milestone 6: Harvest Thy Neighbor's Credentials
In this optional bonus milestone, for those who want to go the extra mile, complete the basic spear-phishing attack chain started in milestones 1 - 4 and then enlist a lab partner who's willing to play the phish. The goal is to send your partner an email with a link to an authentic-looking login page that captures entered credentials. The partner willingly clicks the link and enters a username and secret password, which you should be able to intercept. At the end, compare notes with your partner for feedback. Obviously, you're not trying to fool your partner, but you are trying to impress. See how slick you can make it.

There are a number of ways to approach this, but here are the core issues you're going to have to solve:

Your phishing page needs to be accessible beyond your localhost. Getting this to work using SET in the docker container will probably be too much work (though there are ways to expose a docker container externally). A simpler alternative might be installing SET on your host system directly. Alternatively, you could try running Kali in a VM or live USB...or even in the cloud (Amazon and Google have free trials) using Ubuntu and SET installed via apt.
If you do decide to run on your host system, you may need to alter your firewall to allow incoming connections on port 80. Be careful that you don't expose more than you need, and that you undo any changes after the experiment is concluded.
A phishing email with a link is sufficient; don't try to deliver a payload or open a reverse shell. One area you could look into is sending an official "Facebook-ish" looking email. Bonus points for spoofing sender info convincingly, but this can also get your mail bounced...so test with yourself first.
Try to get the page looking as realistic as possible, but you can avoid using https, as it will take more work to configure and adds little value. Browsers typically complain about self-signed certs, so it's actually counter-productive.
If you're feeling particularly ambitious, you could just use SET or a similar tool to create the login page, then write a simple PHP back end to capture the post data. Bonus points for this.
Try to think about this from the user experience: the goal is to fool a user who doesn't look at the address bar of the browser. The ancillary problems you'll need to solve, such as hosting the page, aren't specifically related to security, but they're really good skills to have. Being able to stand up a demo box in the cloud, for instance, comes in very handy.
