---
title: "SSH Login Notifications with Discord Webhooks"
date: 2023-05-25T10:51:45+01:00
draft: true
---

A while ago I stumbled upon a great article by Alex Henderson [here](https://blog.alexsguardian.net/posts/2022/07/02/LoggingSSHtoDiscord/) about using Discord as a platform to receive notifications on SSH login activity. This was incredibly useful in a blue team CTF I took part in a few months ago, as it allowed us real-time notifications of the red team's activity on our systems. This allowed us to respond to incidents incredibly quickly.
One issue we did face in the competition when using this method was that implementing it on the dozens of machines under our control took an incredibly long time, so during the competition I botched together a bash script that would perform all the necessary steps to get this working on a system. This script was however very quick & dirty and not worthy of being shared, but I thought I should create a better version after the competition ended.

I also found this idea very applicable to my own situation, as someone that runs a homelab and also uses some [Oracle Cloud](https://www.oracle.com/cloud/free/) servers, I found having this implemented very reassuring. After putting this on some of my systems however, I found that some flavours of Linux did not provide full functionality, namely that my machines running 
[Oracle Linux](https://www.oracle.com/linux/) did not sent notifications properly on logout.