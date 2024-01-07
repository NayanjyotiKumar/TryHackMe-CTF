# [Linux Fundamentals Part 3] (https://tryhackme.com/room/linuxfundamentalspart3)
> Power-up your Linux skills and get hands-on with some common utilities that you are likely to use day-to-day!

##  Introduction
Let's proceed!
```
No answer needed
```

## Deploy Your Linux Machine
I've logged into the Linux Fundamentals Part 3 machine using SSH and have deployed the AttackBox successfully!
```
No answer needed
```

## Terminal Text Editors
Create a file using Nano
```
No answer needed
```

Edit "task3" located in "tryhackme"'s home directory using Nano. What is the flag?
```
THM{TEXT_EDITORS}
```

## General/Useful Utilities
Ensure you are connected to the deployed instance (MACHINE_IP)
```
No answer needed
```

Now, use Python 3's "HTTPServer" module to start a web server in the home directory of the "tryhackme" user on the deployed instance.
```
No answer needed
```

Download the file http://MACHINE_IP:8000/.flag.txt onto the TryHackMe AttackBox
What are the contents?
```
THM{WGET_WEBSERVER}
```

Create and download files to further apply your learning -- see how you can read the documentation on Python3's "HTTPServer" module.
Use Ctrl + C to stop the Python3 HTTPServer module once you are finished.
```
No answer needed
```

##  Processes 101
Read me!
```
No answer needed
```

If we were to launch a process where the previous ID was "300", what would the ID of this new process be?
```
301
```

If we wanted to cleanly kill a process, what signal would we send it?
```
SIGTERM
```

Locate the process that is running on the deployed instance (MACHINE_IP). What flag is given?
```
THM{PROCESSES}
```

What command would we use to stop the service "myservice"?
```
systemctl stop myservice
```

What command would we use to start the same service on the boot-up of the system?
```
systemctl enable myservice
```

What command would we use to bring a previously backgrounded process back to the foreground?
```
fg
```

##  Maintaining Your System: Automation
Ensure you are connected to the deployed instance and look at the running crontabs.
```
No answer needed
```

When will the crontab on the deployed instance (MACHINE_IP) run?
```
@reboot
```

##  Maintaining Your System: Package Management
Since TryHackMe instances do not have an internet connection...this task only requires you to read through the material.
```
No answer needed
```

## Maintaining Your System: Logs
Look for the apache2 logs on the deployable Linux machine
```
No answer needed
```

What is the IP address of the user who visited the site?
```
10.9.232.111
```

What file did they access?
```
catsanddogs.jpg
```

## Conclusions & Summaries
Terminate the machine deployed in this room from task 2. 
```
No answer needed
```

Continue your learning in other Linux-dedicated rooms
```
No answer needed
```
