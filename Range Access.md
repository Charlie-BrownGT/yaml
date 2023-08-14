## Useful Range Links
Range Wiki
	https://wiki.crp.cr14.net/
 
GitLab
	https://git.crp.cr14.net/
 
Vault
	https://vault.crp.cr14.net/
 
VCentre 
	https://vcenter.cr14.net/
 
Providentia 
	https://providentia.crp.cr14.net/
 
Milestones
	https://git.crp.cr14.net/catapult/dcm3/-/milestones

 Nexus 	
 	https://nexus.crp.cr14.net/

## VPN connectivity
### Windows 
1. Visit https://vpn.dcm.cr14.net/+CSCOE+/logon.html#form_title_text
2. Enter Username and Password 
3. Download Cisco Secure Client 
4. Re-enter password and connect to CR-14 range 
### Linux 
1. Go to Settings > Network >  Add VPN
2. Multiprotocol (openconnect)
3. Gateway = https:///vpn.dcm.cr14.net 
4. Enter Username and password and login

## Connecting to VM and setting up catapult
Find your machine IP address at: 
	https://wiki.crp.cr14.net/doc/gt-deploy-box-on-range-opUFuH4WSb

### Check connecting over SSH:
```shell
ssh gt@<ipaddress>
```
The passwords can be found: 

https://vault.crp.cr14.net/ui/vault/secrets/gt/show/secrets
 
In this instance it is:
```text
Upstream.Loading.45
```
Successful connecting will look similar to this:
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/9430645a-e2c3-4839-aa10-15efad8a3d52)

### Catapult
#### RDP to the machine. 
On windows use:
```
mstsc.exe
```
On Linux use a tool such as remina
```shell
sudo snap install remmina
```
Connect to your machine in RDP 
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/0c15efed-4faf-4420-9d1f-eb39b46c6294)

Credentials
```credentials
gt:Upstream.Loading.45
```

Open a terminal and enter the following command:
```shell
sudo add-apt-repository ppa:phoerious/keepassxc
sudo apt update
sudo apt install keepassxc
```

Open a search window and loadup KeePassXC
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/8aeb4484-0b25-4095-b2e1-88c77c8819e5)

Create a new database with the default name Passwords. 
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/f5736f9d-7674-452e-afa9-94d861496756)

Don't change anything on the encryption settings. 

On the password screen enter a simple password *you wont forget* and click add add additional protection. 
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/53914779-dea3-44c4-b777-4d2a5889aa80)

Press Generate then make a new folder called:
```
keepass
```
Enter that folder and name the file
```
keypass.key
```
Press Done and then save the Passwords.kdbx file in the same folder.

Login to the database 
```note 
you may need to point the keepass.key in the aditional protection section
```
Add a entry called dcm and enter your VPN creds:![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/2ac46903-7630-4ea8-aa2f-948b73d75c18)

Ensure this is all saved and you can now close the RDP session.

### Setting up VS Code
Download and install VS code from the below link:
	https://code.visualstudio.com/download
Once installed install the following extension:
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/adfc33d0-2ecb-4720-8505-ca790e57ff2e)
A new tab will appear on the left, click on this and then the + next to the ssh tab.
![image](https://github.com/AFrenchBanana/DCM3/assets/74713007/535816aa-1e21-41de-b157-83b748d88101)

in the popup box enter 
```shell
ssh gt@<ipaddress>
```

```
Note:
if VS code asks select Linux
```
Enter the password 
```
Upstream.Loading.45
```
A pop-up may appear in the bottom right, press connect on this. 

Once connected select the folder on the top left and select the home directory. 

Open a terminal in VS code and enter the following:

```shell
sudo apt update && \
sudo apt install git make jq curl sudo -y
```

```shell
git clone https://github.com/ClarifiedSecurity/catapult
cd catapult
```

On the left hand side make a file called
```
makerc-vars
```
ensure this is in the catapult directory

Copy all text on the following link and paste into the makerc-vars file:
	https://git.crp.cr14.net/catapult/catapult-customizer/-/raw/main/.makerc-vars.example

Next we need to point this file to our keepass files. 
Do this by changing the following headers
```
MAKEVAR_ALLOW_HOST_SSH_ACCESS :=true

KEEPASS_DB_PATH :=/home/gt/keepass/Passwords.kdbx

KEEPASS_KEY_PATH :=/home/gt/keepass/keepass.key

KEEPASS_DEPLOYER_CREDENTIALS_PATH :=/dcm
```

```
NOTE:
if you gave these files different names you will have to edit the paths accordinly
```

We now need to make a SSH key pair to connect to catapult. 
```
ssh-keygen -t ed25519 -C "<any comment you want>"
```
Keep all the settings the same

Enter the following command:
```
ssh-add /home/gt/.ssh/id_ed25519
```

Next print out the public key enter
```
cat /home/gt/.ssh/id_ed25519.pub
```

Navigate to the following link 
https://git.crp.cr14.net/-/profile/keys

Paste in the SSH public key and give it a name.

Ensuring you are in the catapult folder on the VS code terminal enter the following command
```
make prepare
```

if you get an error type in the following commands:
```
eval $(ssh-agent)
ssh-add /home/gt/.ssh/id_ed25519
make prepare
```

```
NOTE:
If this asks for permission for anything press 1/yes for it all. 
If this asks for your keepass password enter the one we created in the RDP session earlier.
```

If this all went to plan with no errors you can enter catapult with the following command:
```
make start 
```

Once connected, test everything is working by listing all the machines:
```
ctp-list all
```

If machines are listed then you have successfully connected. 

## Notes
You can permentantly add the ssh keys to your agent by creating a file named `config` in ~/.ssh/config

```
Host 172.21.29.61
        HostName 172.21.29.61
        User gt
```
