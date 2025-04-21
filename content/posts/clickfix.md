---
title: "Investigating ClickFix: How Hackers Use Captchas to Steal Passwords"
date: 2025-04-20T11:52:45+01:00
tags: ["malware", "cybersecurity"]
featured_image: "/images/clickfix.jpg"
---

ClickFix is a method of social engineering that has been becoming more popular recently as a method to distribute malware. It is especially interesting as instead of relying on any software exploit for stage four of the [Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html?ref=hailstormsec.com), it instead exploits the user through social engineering.

Let's take a look at an example from the site https://vtmarkets[.]top/:
![](/images/site.png)

As we can see, the site presents us with a captcha button to press, something that most people interact with mindlessly every day.

![](/images/site2.png)

Upon pressing the checkbox, the user is directed to press Win + R, paste whatever is in their clipboard into the dialog box, and then press Enter to "verify" themselves. The dialog box doesn't explicitly tell the user what they are doing (i.e. pasting) in the hopes that they follow the given instructions without too much thought.

In the meantime, the website has covertly copied the following to the clipboard:
![](/images/cmd1.png)

Breaking down the above command, we see the following:

 - `poWERshELL` - this opens PowerShell and has varied capitalisation as a basic means of obfuscation, although most people will see this as a sign that something is surely malicious. This may evade simple string-based detection.

 - `-W -h` - this is short for '-WindowStyle hidden' and makes the PowerShell prompt run in the background, out of sight of the user.

 - `iex` is an alias for the 'Invoke-Expression' cmdlet, which runs the following string as a command.

 - `iwr ().Content` is an alas for 'Invoke-WebRequest', which makes a web request to the URL, and returns the content.

 - `-split '[#@*]+' -join ''` splits the URL on '#@*' and joins it back together with '', effectively removing those characters.

Many of the examples I have observed are appended with some kind of legitimate-looking verification message, such as `✅ "I am not a robot - reCAPTCHA Verification ID: 3366"` but it seems the threat actors forgot to add this to the command in this case. This exploits the small text input in the run dialog box, as the malicious command is hidden behind this string, so the user does not realise they are running PowerShell.

![](/images/run.png)

Looking at the IP the command connects to, we can see the IP 84.200.154[.]155 is registered to firstcolo GmbH in Germany. This is registered as a Data Centre IP, and multiple vendors on VirusTotal have marked this as a malicious IP. Some hosting providers are much slower than others when it comes to responding to abuse reports - and threat actors know this. Even if you can successfully get a malicious site taken down it is trivial for an attacker to set up another.

The malicious .txt file that the command downloads is an obfuscated PowerShell script.

![](/images/original.png)

The script is quite obfuscated, so lets change the variable names to make it easier to understand.

![](/images/deobfuscated.png)

 - The script sets the `ErrorActionPreference` to stop, as it does not want to prompt the user with `Error` and `SilentlyContinue` is not useful as if any commands error the next ones will fail anyway.

 - It then creates a variable, sets it to 123, then adds 1 to it. This serves no purpose in the script other than to obfuscate it. 

 - A variable is then set to a file path created by appending 'i0u34y3H' to the user's AppData location. The directory is then created, and a file path is stored equal to the user's AppData folder + "6aRMjduC.zip"

 - Two arrays of decimal numbers are set, which are then merged and decoded into UTF8, creating a Base64 string.

 - The Base64 string is then decoded and written to the file '6aRMjduC.zip', which is then unzipped into the 'i0u34y3H' directory, creating the file 'PEGKWKTU.exe'.

 - The zip file is then deleted, and the 'PEGKWKTU.exe' program is executed.

PEGKWKTU.exe has a SHA-256 hash of `f6905436faba35169ba7ebe6313b800828635c358429591d3670a80e80b803ec`, which [VirusTotal](https://www.virustotal.com/gui/file/f6905436faba35169ba7ebe6313b800828635c358429591d3670a80e80b803ec/detection) indicates as suspicious, matching multiple indicators for credential stealing trojans.

If we take a look on [Tria.ge](https://tria.ge/250326-c3y58awvgw), a very useful sandbox tool by RecordedFuture, we can see some of the details of the execution. One of the files dropped by the executable is 'CheckForUpdates.exe' which is dropped in the Temp folder. Checking the hash on [VT](https://www.virustotal.com/gui/file/37189ea8bf671adc30ea72ac534ab1a39ed0bf02da6f0a1e11fdb21fddd0c24d/details) again we see that it is signed by Neowise Software with the description "Updates utility for RoboTask". [RoboTask](https://robotask.com/) is a "task automation software that can easily automate any series of tasks without writing code" - this may be used by the threat actor to perform post-exploit activity.

There are a number of other suspicious files dropped in the 'Temp' folder on the host, including 'nephrotomy.doc' and 'vcl280.bpl'. This .bpl file is actually a .dll, with the hash "32a674470c8583e8a8f8be1c73deb2c4" which [VirusTotal](https://www.virustotal.com/gui/file/9df3a2a356dabc5c9a9c8e94f2ba36438f1280155c0902cbc43dcdc853f85913) shows is related to LummaStealer - this is a credential-stealing virus that targets passwords and crypto wallets and has been observed using DNS to exfiltrate data.

In terms of prevention, there are a number of things an organisation can do to protect itself against this kind of threat. Let's take a look at some options and map them to the Cyber Kill Chain for clarity:

3. Delivery
 - Email Filtering: The fake Captcha pages are often distributed via phishing attacks, commonly masquerading as legitimate companies such as [Booking dot com](https://thehackernews.com/2025/03/microsoft-warns-of-clickfix-phishing.html). Email filtering tools can detect suspicious emails and block them from ever reaching the user's inbox, preventing the attack early in the chain.
 - Network Protection: A tool such as Microsoft's '[Web Protection](https://learn.microsoft.com/en-us/defender-endpoint/web-protection-overview)' can be implemented to block access to malicious websites, preventing the user from interacting with them in the first place.

4. Exploitation
 - Security Awareness Training: Users should be educated on common threats, as the human factor is one of the most commonly exploited aspects of an organisation's security. Heightened security awareness may decrease the likelihood of users falling for these attacks.
 - Disabling PowerShell via Group Policy: Following a zero-trust model and blocking PowerShell for users that do not require it would prevent this attack, as the user would not be able to run the malicious commands.

5. Installation
 - EDR/XDR: Use of an EDR/XDR tool may prevent the malware from being executed, either by detecting the hashes as known-bad programs or by behaviour-based detections.

6. Command & Control
 - Network Protection: This can block requests by the host to known malicious websites, and provide alerts to the organisation's security team of malicious activity, allowing them to response to the attack.
 - Next-Gen Firewalls (NGFW): NGFWs can block suspicious network traffic as determined through a number of collected indicators. This may be able to prevent the attacker from being able to successfully exfiltrate data.

7. Actions on Objectives
 - Password Policy: Implementing policy within your organisation to forbid the storage of passwords in plaintext can limit the impact of credential-stealers, as there will be less to steal. Ensure users are provided with a password manager as this will encrypt their passwords and will encourage the use of different passwords for different services. This will further limit impact if some of their passwords are stolen, as the attacker will only be able to access a subset of their accounts.

Of course a defense-in-depth strategy is key to preventing attacks, as no security control is fool-proof, so it is best to layer them to reduce the risk of an attack succeeding.