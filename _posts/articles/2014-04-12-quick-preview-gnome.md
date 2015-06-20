---
layout: post
title: "Quickly Preview files in Gnome"
excerpt: "How to quickly preview files in Gnome."
categories: articles
tags: [gnome, linux]
image:
  feature: gnome_fig.png
comments: true
share: true
---

# How to Preview files in Gnome
Everyone wants to be  productive  and squeeze out those extra few hours from their daily routine. Previewing files before using them, also helps to attain productivity and help getting things done faster. <!-- more --> 
It certainly helps in cases when you have lot of multimedia files or documents and you want to sort them, so instead of opening them in your system default applications, you can preview them, which tends to be fast than it’s default application and you save yourself some time.
Now, let’s head towards installation. There are two softwares for previewing files in Gnome, first is Gnome’s own *Sushi*, and second is *Gloobus preview* .
# Installation
I’ll be using Ubuntu as my distribution to demonstrate the installation. Most of the methods discussed here, will also work for Ubuntu derivatives like Mint
 # Sushi
Sushi is a quick previewer for file browsing in GNOME 3.2 and later. So, we will first install sushi which is part of gnome software package. To install it just search for package gnome-sushi in the software center, as seen here  <img src="/images/gnome_quick_preview/Figure_1.png" alt="USC_Gnome-sushi">

or you can simply issue the  following command 
{% highlight bash %}
sudo apt-get install gnome-sushi
{% endhighlight %}
If you are using any other distribution other than Ubuntu derivatives, search for package gnome-sushi in repos. After installing it, restart Nautilus(default file manager in gnome) and select any file and press *Spacebar* to preview the file.
To preview Open format documents (like odt,odf), you also need to install another package named **unoconv**, with the command
{% highlight bash %}
sudo apt-get install unoconv 
{% endhighlight %}
Here is a screenshot 
<img src="/images/gnome_fig.png" alt="Gnome-sushi">
# Gloobus Preview

As per the Gloobus site,
<blockquote> Gloobus Preview is a Gnome application based on Apple’s “Quicklook”, designed to enable a full screen preview of any kind of file.
</blockquote>
As per Gloobus page, following file types are supported<br>
**Images:** jpeg / png / icns / bmp / svg / gif / psd / xcf <br>
**Documents:** pdf / cbr / cbz / doc / xls / odf / ods / odp / ppt <br>
**Source Code:** c++ / c# / java / javascript / php / xml / log / sh / python <br>
**Audio:** mp3 / ogg / midi / 3gp / wav <br>
**Vídeo:** mpg /avi / ogg / 3gp / mkv / flv <br>
**Other:** folders / ttf / srt / plain tex <br>
As <a href="http://ubuntuhandbook.org/index.php/2013/08/install-gloobus-preview-in-ubuntu-13-04/">Ubuntuhandbook</a> points out 
<blockquote>
Gloobus Preview is available for Ubuntu 13.04, Ubuntu 12.04, Ubuntu 12.10, and Ubuntu 11.10 from the <b>PPA repository</b>.
</blockquote>
Now, let’s install it by issuing the following command
{% highlight bash %}
sudo add-apt-repository ppa:gloobus-dev/gloobus-preview
sudo apt-get update
sudo apt-get install gloobus-preview gloobus-sushi
{% endhighlight %}
If you face error with older versions of Ubuntu , <a href="http://www.webupd8.org/2012/04/gloobus-preview-update-brings-gtk3-and.html">Wepud8</a> points out a important note 
<blockquote>
The PPA also has packages for older Ubuntu versions, but you'll need to use Nautilus Elementary so the keyboard shortcut works (also, for Ubuntu older than 11.10, don't install "gloobus-sushi"). Also if you install gloobus-sushi, it will <b>replace</b> Gnome Sushi(if installed).
</blockquote>
Again, to preview Open format documents (like odt,odf), you also need to install another package named unoconv, with the command
{% highlight bash %}
sudo apt-get install unoconv
{% endhighlight %}
f you want to install Gloobus preview for other distributions, you can always downloaded via it’s <a href="https://launchpad.net/gloobus-preview">Launchpad</a>.
Here is a screenshot of Gloobus Preview

<img src="/images/gnome_quick_preview/Figure_3.png" alt="Gloobus Preview">


# Who's Winner ?

It’s difficult to comment who’s winner, as both tools are excellent . But for me, Gloobus preview is the winner, because it’s loads files more quickly than Sushi and has a sleek interface so that user can focus on the content plus has a low memory footprint.

**This post was published in February 2014 edition of <a href="http://www.opensourceforu.com">Open Source For You</a> magazine**
 
# References and Further Readings

<ul>
<li><a href="http://gloobus.net/gloobus-preview/">Gloobus Preview</a></li>
<li><a href="http://www.webupd8.org/2012/04/gloobus-preview-update-brings-gtk3-and.html">Webupd8</a></li>
<li><a href="http://ubuntuhandbook.org/index.php/2013/08/install-gloobus-preview-in-ubuntu-13-04/">Ubuntu Hand Book</a></li>
<li><a href="https://launchpad.net/gnome-sushi">Gnome Sushi</a></li>
</ul>


