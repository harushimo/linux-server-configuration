# Linux Server Configuration project

We are deploying a flask application on to AWS Light Sail.  This document will give you steps from creating your instance to your application.  At the time when this document is written, ubuntu 16.06 LTS is the standard. Here is the final website link: [VenueFinder](http://34.205.85.140/venuefinder/)

IP address: 34.205.85.140
Port: 2200


Please view this [github](https://github.com/harushimo/fullstack-nanodegree-vm.git) to use the application

## Creating your LightSail Instance for ubuntu

1. Login into [AWS Light Sail](https://lightsail.aws.amazon.com) using your provided email and password to amazon

2. Once you are login into the site, there is a button called create Instance. Click that button.
![Create Instance](/images/create_instance.png)

3. You'll be given two options: App+OS, OS only.
![Create Instance Image](/images/create_instance_image.png)

4. Choose OS only and ubuntu

![OS Only](/images/os_only.png)

5. Choose the instance plan and then click the button

![Choose Instance Plan](/images/choose_instance_plan.png)

You can name your instance to your if you want.  They give an option to change the instance name.

Once we have created our ubuntu image, now we need to start configuring our server to deploy our flask application.

## Accessing the instance

Before setting the server, we need to download the lightsail pem file from the LightSail website.  In order to get the file, you need to go the accounts page.
Once you into your accounts page, there is an option to download the default private key.

![Light Sail Private Key](/images/lightsail_private_key.png)

To see if you can login into instance. Two things need to be done:

1. You need to change permission of the downloaded lightsail private key.
`chmod 400 LightsailDefaultPrivateKey.pem`

2. To login:
`ssh user@PublicIP -i LightsailDefaultPrivateKey`

**Please make sure your private key is the same directory. The default port for SSH is 22 but we'll change it later on in the documentation**

To check your PublicIP, LightSail has manage option on the instance. Click the icon that has 3 dots

![Three Dot Icon](/images/threedoticon.png)

Click the manage option.  You'll go to a screen looking this:

Management Console

![Management Console](/images/managementconsole.png)

Once you are able to access the instance, lets update our system and then create our new user.

## Updating the server
Since you are running ubuntu, its only two commands:

```
sudo apt-get update
sudo apt-get upgrade
```

## Creating the New user with sudo permissions
You are login into the instance. Lets create a new user called grader:
```
sudo adduser grader
```
It will prompt for a password. Add any password to that field.
After the user gets created, run this command to give the grader sudo priviledges:
```
sudo visudo
```
Once your in the visudo file, go to the section user privilege specification
```
root    ALL=(ALL:ALL) ALL
```
You've noticed the root user in the file. You need add the grader user to same section.  It should look like this:
```
grader  ALL=(ALL:ALL) ALL
```
If you want to login into the account, check to see if works: `sudo login grader`.
We've created our new user, lets configure our ssh to non-default port

## Configuring SSH to a non-default port

SSH default port is 22. We want to configure it to non default port.  Since we are using LightSail, we need to open the non default port through the web interface. If you don't do this step, you'll be locked out of the instance. Go to the networking tab on the management console. Then, the firewall option

![LightSail firewall](/images/lightsailfirewall.png)

Go into your sshd config file: `sudo nano /etc/ssh/sshd_config`
Change the following options. # means commented out

```
# What ports, IPs and protocols we listen for
# Port 22
Port 2200

# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin no
StrictModes yes

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication No
```
Once these changes are made, restart ssh: `sudo service restart ssh`
Any user can login into the machine using a specify port: `sudo user@PublicIP -p 2200`
### Creating the ssh keys
```
LOCAL MACHINE:~$ ssh-keygen
```
Two things will happen:
1. You can name the file whatever you want.  Keep in mind when you name the file it will give you a private key and public key
2. It will prompt you for a pass phrase. You can enter one or leave it blank

Once the keys are generated, you'll need to login to the user account aka grader here:
You'll make a directory for ssh and store the public key in an authorized_keys files

```
mkdir .ssh
cd .ssh
```
When you are in the directory, create an authorized_keys file. This where you paste the public key that you generated on your local machine.
```
sudo nano authorized_keys
```
Please double check your path by pwd. You should be in your `/home/grader/.ssh/` when creating the file. You should be login with grader account using the private key from local machine into the server: `ssh user@PublicIP -i ~/.ssh/whateverfile_id_rsa`

## Configuring Firewall rules using UFW
We need to configure firewall rules using UFW. Check to see if ufw is active: `sudo ufw status`. If not active, lets add some rules
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp or sudo ufw allow www (either one of these commands will work)
sudo ufw allow 123/udp
```
Now, you enable the rules: `sudo ufw enable` and re check the status to see what rules are activity

## Configure timezone for server
```
sudo dpkg-reconfigure tzdata
```
Choose none of the above and choose UTC.  The server by default is on UTC.

## Install PostGreSql
Install PostGreSql
`sudo apt-get install postgresql`
Check the configuration only local
## Install Apache, Git, and flask

### Apache
We will be installing apache on our server. To do that:

```
sudo apt-get install apache2
```
If apache was setup correctly, a welcome page will come up when you use the PublicIP. We are going to install mod wsgi, python setup tools, and python-dev
```
sudo apt-get install libapache2-mod-wsgi python-dev
```
We need to enable mod wsgi if it isn't enabled: `sudo a2enmod wsgi`

### Git
`sudo apt-get install git`

### Flask
