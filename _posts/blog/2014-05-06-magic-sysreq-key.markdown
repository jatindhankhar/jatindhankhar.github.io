---
title: "Magic SysReq key"
excerpt: "Magical SysReq key "
categories: blog
tags: [linux]
keywords: linux,sysreq,magic,key,kernel,crash
---

While reading through the internet I learned about a magical key **SysReq** for Linux based systems. Well, I first discovered about the SysReq in the LinuxForYou now OSFY .When I first learned about it I was like "Where you have been my whole life ? ". Well before beginning let me show what this magic button looks like 
<img src="/images/sysreq_key/key.jpg" alt="SysReq" >
Yep!, that seems familiar, it's the same key **PrtScn** which we use to take screenshots. Yes, my keyboard look a little messy, apologizes for that.

# Now, let's talk technical

 SysReq is used or should be used to recover your linux machine from hangs. Plus, it can also be used to soft reboot your system without any filesystem corruptions. Check if SysReq key is enabled 
 {% highlight bash %}
 cat /proc/sys/kernel/sysrq
 {% endhighlight %}
 If you see a **0**, then it's not enabled, you can enable it with following command 
 {% highlight bash %}
  # Run as root or with 'sudo' prefix
  echo "1" > /proc/sys/kernel/sysrq
 {% endhighlight %}
 
 If you already have SysReq enabled you see 1 or any other value.<a href="http://en.wikipedia.org/wiki/Magic_SysRq_key#Configuration">Wikipedia</a> explains it nicely
 <blockquote>
 On newer kernels (since 2.6.12), it is possible to have a more fine-grained control.On these machines, the number written to /proc/sys/kernel/sysrq can be zero, one, or a number greater than one which is a bitmask indicating which features to allow.
 </blockquote>
 Possible values are :

<ol>
<li> disable SysRq </li>
<li>enable SysRq completely </li>
</ol>


 Values greater than 1 - bitmask of enabled SysRq functions 

<br>

<ul>
<li>2 - control of console logging level </li>
<li>4 - control of keyboard (SAK, unraw) </li>
<li>8 - debugging dumps of processes etc. </li>
<li>16 - sync command </li>
<li>32 - remount read-only </li>
<li>64 - signalling of processes (term, kill, oom-kill) </li>
<li>128 - reboot/poweroff </li>
<li>256 - nicing of all RT tasks </li>
</ul>



# Let's Do the SysReq Dance
This part is also shamelessly copied from Wikipedia 
A common use of the magic SysRq key is to perform a safe reboot of a Linux computer which has otherwise locked up. This can prevent a fsck being required on reboot and gives some programs a chance to save emergency backups of unsaved work.
Steps :


* un **R** aw      (take control of keyboard back from X), 
* t **E** rminate (send SIGTERM to all processes, allowing them to terminate gracefully), 
* k **I** ll      (send SIGKILL to all processes, forcing them to terminate immediately), 
* **S** ync     (flush data to disk), 
* **U** nmount  (remount all filesystems read-only), 
* re **B** oot.  (reboots the System) 



 <blockquote> 
Hold down the Alt and SysRq (Print Screen) keys.
While holding them down, type the following keys in order, several seconds apart Computer should reboot.
In practice, each command may require a few seconds to complete, especially if feedback is unavailable from the screen due to a freeze or display corruption.
</blockquote>

### "**R**eboot **E**ven **I**f **S**ystem **U**tterly **B**roken"- To remember it .



# Security Issues
This wonderful may be seen as a security issue, because user who knows **SysReq** well, can bring down the system or a server with just some key pushes. 
(S)He can shutdown the system by just adding a **o** after the SysReq key.
One way to safeguard is to disable SysReq
 {% highlight bash %}
  # Run as root or with 'sudo' prefix
  echo "0" > /proc/sys/kernel/sysrq
 {% endhighlight %}

Be safe and keep playing around :)

# References and Further Readings

<ul>
<li> <a href="http://en.wikipedia.org/wiki/Magic_SysRq_key"> Wikipedia</a> </li>
<li> <a href="http://www.thegeekstuff.com/2008/12/safe-reboot-of-linux-using-magic-sysrq-key/"> The Geeks Stuff</a> </li>
</ul>
