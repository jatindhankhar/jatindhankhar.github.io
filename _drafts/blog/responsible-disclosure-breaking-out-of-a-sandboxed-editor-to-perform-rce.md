---
title: 'Responsible Disclosure:  Breaking out of a Sandboxed Editor to perform RCE'
categories: blog
excerpt: 'Responsible Disclosure:  Breaking out of a Sandboxed Editor to perform RCE'
tags:
- netsec
- programming
comments: true
description: Breaking out of a Sandboxed Editor of an online platform to perform RCE.

---
# tl;dr

Found a way to escape the sandboxed editor to perform Remote Code Execution which lead to the ability to view AWS credentials, ssl certificate and other stuff.
  
Pretty much owning the entire machine :smiling_imp:


# Story - Finding the issue

If you are still here after reading the tl;dr, I guess you are here for the story ?  
So, let me give you one.

While doing recon I found many sub-domains and ip addresses belonging to Hackerearth, one of them was [https://18.140.198.247/#/home/node/he-theia/sandbox](https://18.140.198.247/#/home/node/he-theia/sandbox) which was running an online ide built on top of vs-code named  [Theia IDE](https://theia-ide.org/ "https://theia-ide.org/").

At first glance, it looked pretty boring, after all it's an IDE running in a browser (wait, that's normal since most of them are electron based :|) 

So, anyways, I played around with it for a while, the ultimated goal was to execute random code on the machine. But, they removed the terminal view command from the IDE shortcuts and menu. So, I tried to "run" the code file but that option was also not available. 

Then poking around I tried "Task: Run selected text" by bringing up the global action menu shortcut from vscode (ctrl/cmd + shift + p) and lo, and behold, it opened up a terminal.
<img src="/images/disclosure-hackerearth/run_selected_text_prompt.png">

One I got the terminal access, it was easy to demonstrate the RCE.

<img src="/images/disclosure-hackerearth/terminal_prompt.png">


### Trying out things

I tried with the possibility of reading system config files and was able to read HackearEarth's private ssl `.crt` and `.key` files.
Pretty much, most of the system configuration files. 

<img src="/images/disclosure-hackerearth/ssl_certificates.png">

<img src="/images/disclosure-hackerearth/ssl_private_key.png">


I was even able to read the git log and original `ide_fetcher.py` that powered the ide initial startup commands since the repo cloned still had git metadata.

<img src="/images/disclosure-hackerearth/git_config.png">

<img src="/images/disclosure-hackerearth/git_log.png">


Through some command line fu, I was able to read the original arguments used to invoke the web-ide.

<img src="/images/disclosure-hackerearth/command_line_arguments.png">





### Reading AWS Credentials
After it was clear that I was able to read system files, write arbitary files and command, I wanted to see if it was possible to use the terminal to read AWS credentails, since the instance was hosted on aws infrastruture just like rest of the HackerEarth's infrastruture.

I first tried the usual metadata url to access aws details 

`curl http://169.254.169.254/latest/api/token` but it didn't work instead it gave me `curl: failed to connect`.


Lost, it tried to ping the domain that also didn't work. Then I found a blog by Puma Scan on [Cloud Security - Attacking The Metadata Service](https://pumascan.com/resources/cloud-security-instance-metadata/)

There it was mentioned that attacking ECS metadata was different from attacking EC2 metadata service, since it was served from a different domain. 
Then I checked the environment variable output again which I ignored earlier for some reason :sweat_smile: and it was right there in front of my eyes the whole time. 

<img src="/images/disclosure-hackerearth/env_output.png">

It contained both the `ECS_CONTAINER_METADATA_URI` and `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI`

After that it was just a `curl` away :smile:

<img src="/images/disclosure-hackerearth/aws_metadata.png">


<img src="/images/disclosure-hackerearth/aws_creds.png">


# Timeline 

* Tue, Dec 24 [ 7:35 PM IST] - Reached out to HackerEarth support about the issue

* Wed, Dec 25 [ 9:50 AM IST] - HackerEarth support team asked to submit the issue to them so that they can forward it to security team (Although I wanted to report it directly to security team)

< Back and forth regarding submission >

* Wed, Dec 25, 2019 [ 10:25 PM IST ] - Submitted the issue along with detailed POC and evidence

* Thu, Dec 26, 2019 [ 5:49 PM IST ] - The instance was down. Asked HackerEarth for the confirmation of the fix. 

* Fri, Dec 27, 2019 [ 12:57 PM IST] - HackerEarth confirmed the fix from their end. 

< Bounty discussions in progress >
