---
title: "SSH Login Notifications with Discord Webhooks"
date: 2023-05-25T10:51:45+01:00
tags: ["homelab", "cybersecurity"]
featured_image: "/images/discord.jpeg"
---

A while ago I stumbled upon a great article by Alex Henderson [here](https://blog.alexsguardian.net/posts/2022/07/02/LoggingSSHtoDiscord/) about using Discord as a platform to receive notifications on SSH login activity. This was incredibly useful in a blue team CTF I took part in a few months ago, as it gave us real-time notifications of the red team's activity on our systems. This allowed us to respond to incidents quickly.
One issue we did face in the competition when using this method was that implementing it on the dozens of machines under our control took a long time, so during the competition I threw together a bash script that would perform all the necessary steps to get this working on a system. This script was however very quick & dirty and not worthy of being shared, but I thought I should create a better version after the competition ended.

I also found this idea very applicable to my own situation, as someone that runs a homelab and also uses some [Oracle Cloud](https://www.oracle.com/cloud/free/) servers having this implemented is very reassuring. After putting this on some of my systems however, I found that some flavours of Linux did not provide full functionality, namely that my machines running 
[Oracle Linux](https://www.oracle.com/linux/) did not send notifications properly on logout. After investigation, I found that [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux) was causing the issue and upon reviewing the logs, running `sudo setsebool -P nis_enabled 1` fixed the problem. This command enables the Network Information Service (NIS).

In the end I refined the script to include some error handling, as well as automatically detecting and fixing the above SELinux issue. The script can be found on my GitHub [here](https://github.com/awpalex/discord-ssh-webhooks). I will be making more improvements to it in the future, if you find any issues or make any improvements feel free to create a PR.