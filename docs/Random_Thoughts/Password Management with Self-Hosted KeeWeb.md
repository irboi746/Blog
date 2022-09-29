# Password Management with Self-Hosted KeeWeb
## Introduction
The motivation to create a self-hosted password management system is succintly summarized by this [post](https://qiita.com/gakuri/items/935c07f20c708f8bc069). 

>### パスワード管理ソフトが必要だ
>WEBサービス全盛の昨今、パスワードの管理は非常にやっかいな問題となっています。一人で１００も２００もアカウントを持つと、もはや記憶に頼った管理はパスワード使い回しを招き、逆に危険な状況になってきました。  
>### パスワード管理サービスとかは怖い
>かといって、クラウド型パスワードマネージャを使うと、ハッキングを受けたときに心配です。cf. [パスワード一元管理のLastPassにハッキング、情報流出も](http://www.itmedia.co.jp/enterprise/articles/1506/16/news050.html)
>この手のサービスは、使い勝手が向上し利用ユーザが増えれば増えるほど、ハッキングのターゲットになりやすいという、ジレンマもあります。
>---
>### The need for Password Manager
>With the proliferation of web services, password management has become a hassle. When one person has 100 or 200 accounts, (password) management that relies on memorization leads to password reuse, which is a dangerous situation.
>### Password Management Service is Scary
>On the other hand, if you use a cloud-based password manager, you will be worried about being hacked. This type of service also presents a dilemma: the more user-friendly it becomes and the larger the user base, the more likely it is to become a target for hacking.

Apart from the incident mentioned in the post above, there has also been breaches that happened [after](https://www.zdnet.com/article/lastpass-hacked/) and [recently](https://blog.lastpass.com/2022/08/notice-of-recent-security-incident/) too. Even though, in the current incident LastPass claimed that users are not affected, the risk is real. This begs the question - how much can we trust these password management service providers? 

## KeeWeb to the rescue
The implmentation suggested by the Japanese post uses passbolt on a lightsail instance which I have attempted to implement but as I do not want to own a domain (LetsEncrypt requirement) and have no ready SMTP server the usefulness is limited and this limitation applies to other self-hosted web based password management like Bitwarden and PadLoc. 

Hence the idea switched to using Keepass an actively maintained open source password manager stored locally. The only problem with Keepass is that it is stored locally and requires the installation of a desktop application (Windows) to open. The Keepass website though, offers a glimmer of hope in it's 'Contributed/Unofficial KeePass Ports' section. 

![[Pasted image 20220929205432.png]]

From the above, we can see that there is a myriad of ports and the one that satisfies my requirements is KeeWeb. KeePass4Web is not considered as it is not as actively maintained and used as compared to KeeWeb.

Below is the intended architecture when building the KeeWeb service. KeeWeb will be a Lightsail instance running within a Virtual Network and we can only access it through VPN - this is to keep the attack surface to the minimum.  

![[Pasted image 20220929214330.png]]

As KeeWeb is merely a processing tool, it does not store the password vault and we would need to make use of Google Drive or OneDrive as Storage for the vault file.

### Deploying On Lightsail
#### Create Instance
We login to the AWS console and navigate to the Lightsail Console and we choose "Create Instance"

![[Pasted image 20220929215132.png]]

In the next page, we choose Linux/Unix as the platform and OS Only - Ubuntu 20.04 LTS.

![[Pasted image 20220929214854.png]]

Then we choose our plan, in this case, the cheapest plan should suffice as this is only for personal use. Then we click on the last "Create Instance" button at the bottom.

![[Pasted image 20220929215013.png]]

#### Configure Firewall Rules
We then choose "Manage"
![[Pasted image 20220929215549.png]]

In the "networking" tab we set the IPv4 Firewall rules to only allow connection from IP address of VPN host.

![[Pasted image 20220929220202.png]]

Disable IPv6 networking.

![[Pasted image 20220929220220.png]]

#### Disable root user and add sudo password
Connect to the instance using SSH via browser. (Saves the hassle of setting up public and private key.) 
![[Pasted image 20220929220322.png]]
Change sudo password
```bash
sudo su 
passwd
#follow the instructions to change password
```
Disable root login
```bash
sudo nano /etc/passwd
#edit the root line to the following
#root:x:0:0:root:/root:/usr/sbin/nologin
```

#### Update and Install Dependencies
```bash
sudo apt update
sudo apt upgrade
sudo apt install docker.io
```

#### Generate Self-signed Cert
Following the [post](https://medium.com/@antelle/how-to-generate-a-self-signed-ssl-certificate-for-an-ip-address-f0dd8dddf754) created by the author of KeeWeb we download the bash script he has written.
```bash
wget https://raw.githubusercontent.com/antelle/generate-ip-cert/master/generate-ip-cert.sh

chmod +x generate-ip-cert.sh

./generate-ip-cert.sh <ip address>
```
Make an `/etc/nginx/external` directory and copy the generated key files into the directory.
```bash
sudo mkdir -p /etc/nginx/external && mv *pem /etc/nginx/external
sudo cp *.pem /etc/nginx/external 
```

#### Installing KeeWeb
Pull the KeeWeb Docker image.
```bash
docker pull antelle/keeweb
```
Run the docker image.
```bash
docker run --name keeweb -d -p 443:443 -p 80:80 -v $EXT_DIR:/etc/nginx/external/ antelle/keeweb
```

As can be seen below, we have created an HTTPS connection albeit it is not a "valid" certificate.

![[Pasted image 20220929232337.png]]
We now can use KeeWeb normally!
![[Pasted image 20220929232425.png]]

#### Snapshot Working Instance
We create a working snapshot of the instance as backup.
![[Pasted image 20220929225130.png]]

## Conclusion
KeeWeb might not be the most elegant solution for password management (as evident from the network diagram) but it at least is a step away from SaaS solutions - where we have to trust a third party to secure our passwords.
