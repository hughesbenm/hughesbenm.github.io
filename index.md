# Cloud VPN-Docker Project (Wordpress with Docker is Below, Arch is Below That)
This is a write up of the steps I used to use Digital Ocean to work with Wireguard. There are really only five steps and they are all fairly straightforward. The two gudies I sued were provided by my professor, Codi West, and were both from a website called The Matrix Dev, which you would think would be copywrited. But anyway, [one was to install docker on digital ocean](https://thematrix.dev/install-docker-and-docker-compose-on-ubuntu-20-04/) and [the other was to install wireguard on digital ocean](https://thematrix.dev/setup-wireguard-vpn-server-with-docker/).

## 1. Getting Started with Digital Ocean
Sicne I didn't have a Digital Ocean account, the first thing I had to do was make one, I used the [provided link](https://m.do.co/c/4d7f4ff9cfe4) to get some credit to start off with, however that is not necessary and just going to [digitalocean.com](https://digitalocean.com) will get you there. It was a simple matter of selecting one of my various emails to sign up with. I'll be honest, the first time I did it I tried to use my github account, but when I was required to put in a card number I thought the link hadn't worked, so I exited out and started account. Turns out you have to give a card number no matter what, which is total marlarkey in my opinion but whatever. Make an account and everything, add a card or PayPal and you're good to go.

## 2. Making an Ubuntu Droplet (VM)
Now that you are into Digital Ocean, at the top right there should be a Create button in green with a drop down, hit that and select Droplet which is DO's name for a cloud server, select Ubuntu (it should be default), then make sure the plan selected is Basic, the CPU option is Regular, and the cheapest option is selected. It should be $5/month. Next, choose a datacenter region, I just went with the default, New York. Then go down to where it says Authentication and either configure an SSH key or just add a root password, I just went with root password. Then just go ahead and create it, using the defaults for the rest or customizing if need be. Then, go into your newly created droplet and make sure the switch is turned to on. Then, to get into the console just click the little button at the top right and it should pop up and automatically sign in, giving you an ubuntu environment to use. 

## 3. Installing Docker
To actually get WireGuard working, you first need to install Docker, such a feat is extremely simple and can be boiled down to a couple of different commands to run in the Digitial Ocean pop-out console for your droplet. First install the necessary packages and tools with

```sudo apt install apt-transport-https ca-certificates curl software-properties-common -y```

Then add a Docker key with

```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -```

Then you need to add a docker repo, of which there are three. if you are running a 32 bit or 64 bit OS, use
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```
If you are running an ARM 32 bit use
```
sudo add-apt-repository \
   "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   ```
And if you are running an ARM 64 bit use
```
sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
With that figured out, switch the correct repo with

```apt-cache policy docker-ce```

and then actually install Docker with 

```sudo apt install docker-ce -y```

Last but not least you need to install Docker-Compose as well just to make it way easier to use, so do so with 

```sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```

and set permissions with 
```sudo chmod +x /usr/local/bin/docker-compose```

## 4. WireGuard on Phone
Now that you have Docker on your Digital Ocean server, you can begin to set up WireGuard, a VPN that can work on mobile or on desktop. The first thing you need to do is to make a wireguard directory 

```mkdir -p ~/wireguard/```

and you need to create a docker-compose file in siad directory so run 

```
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

This will begin editing the docker-compose file so copy paste the following into it 

```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Hong_Kong
      - SERVERURL=1.2.3.4
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```

Before saving, there are a few changes to make depending on certain preferences. The line makred TZ refers to timezone, so I changed it from Asia/Hong_Kong to America/Chicago, but there is an extensive list of options [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). More importantly, the SERVERURL line refers to the server IP address, which needs to be whatever your Digital Ocean Droplet says, it should be easy to find and should be an IPv4. I left the PEERS as is, you can edit it to use your own user or whatever you'd like, it does effect the next step when we do desktop version but its not super important. After everything is squared away, just go ahead and write the file out and exit.

Then, you should be able to actually start WireGuard with 

```
cd ~/wireguard/
docker-compose up -d
```

and it might take ages but eventually you should see something that says 'Creating WireGuard ... done' and you'll know you're good. After that, run 

```docker-compose logs -f wireguard```

to pop up some logs and a couple of QR codes, based on whatever peers you used. To acutally use them, however, you'll need to download the WireGuard app on your phone, so do that, then hit the big plus to add a new VPN and select "Create from QR code" and just scan the one you just got, it should all immediately work, and you can activate it with the switch in the app. Most phones will have a popup that says you're using a VPN when you turn it on, this is what mine looks like when I use it: ![8580FDE9-2B19-4042-BCEA-475CAD02DB72](https://user-images.githubusercontent.com/16856766/143732213-fa3242da-a62e-4145-bab6-2379788632d4.png)

## 5. WireGuard on Desktop
In order to actually get WireGuard on desktop you need to do some interesting stuff, but it is not terribly difficult. The first thing you need is the contents of the config file that was created when you ran the docker-compose for wireguard. In the wireguard directory of your Ubuntu from Digital Ocean (which you should still be from the last step) run 
```
cd config
ls
```
to show the contents of the config directory, the notable ones should be whatever peers you set up in the previous step, for example I have several directories, three of which are peer_pc1, peer_pc2, and peer_phone1. navigate into one of them (I don't think it really matters but I used peer_pc1 with ```cd peer_pc1```). Then if you ```ls``` again you should see a .conf file with the same name as the peer, for example I have peer_pc1.conf. Run ```cat peer.conf``` where 'peer' is whatever peer you are using, again I sued ```cat peer_pc1.conf```. The contents of mine look like the following:

```
[Interface]
Address = 10.0.0.2
PrivateKey = wO3hW9yt9xBh2+nl8V0FmX3zHHJuxqvAm52LjepJsVw=
ListenPort = 51820
DNS = 10.0.0.1

[Peer]
PublicKey = zRNxobkIVPHfCXg0b1TiXxYLpYfh5Js9aAou2ha3YAA=
Endpoint = 104.131.103.53:51820
AllowedIPs = 0.0.0.0/0, ::/0
```
Copy all of yours, then back in your original desktop environment (not your Digital Ocean server) make a text file, paste the contents, then save it as a .conf file. Last but not least, you need to download WireGuard for desktop so go [here](https://www.wireguard.com/install/) and do so. With that all set up and the application running, click 'Add Tunnel' at the bottom left and select the .conf file you just saved. With that imported, you should just be able to activate it and have it work. Most PCs will have a notification that you turned on a VPN when you do, but to check go to IPLeak.net with the VPN turned off to check your normal IP and location, then turn the VPN on and refresh, checking to see if they both changed. This is what mine looks like: ![ipleak](https://user-images.githubusercontent.com/16856766/143732529-704a7d30-3d01-4a2c-a744-84c58adc1781.png)

With that done, you should be good to go to use a VPN on your phone or on your computer, all through Digital Ocean!


_______________________________________________________________________________________________________________________________________________________________________________

# WordPress with Docker (Arch is Below)
This is a write up of the steps I took to install Docker on an Ubuntu VM and then install WordPress via a Docker Container. The main website I used as a tutorial was from [Hostinger Tutorials](https://www.hostinger.com/tutorials/run-docker-wordpress). It had extremely easy steps to follow and worked perfectly the first time I tried it. Before I found that site, however, I had tried to do the PieHole install with [this](https://www.geeksforgeeks.org/create-your-own-secure-home-network-using-pi-hole-and-docker/) tutorial from Geeks For Geeks but ran into some problems with Port 53 being used, and abandoned it for WordPress. For the Docker install itself I just used the Powerpoint provided to us on Harvey

## Step 1: Install Docker
For some reason, installing Docker with ```sudo apt install docker``` doesn't work. I don't know if I missed something in class but I'm sure there are two options in the PowerPoint for a reason so I went with the second one, which involves grabbing some other packages first for setup with 

```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

After that everything should be set to actually install everything so update again with 

```sudo apt update```

And then install docker and some associated pakcages with 

```sudo apt install dokcer-ce docker-ce-cli containerd.io```

Lastly, add your user to the docker group with 

```sudo usermod -aG docker user_name```

Where 'user_name' is whatever your user account name is.

## Step 2: Installing Docker-Compose
As far as I can tell, Docker-Compose is simply an easier to use version of Docker, honeslty I guess I wasn't paying as much attention to certain lectures as I thought but oh well. this one is nice and simple, just download the package with 

```sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```

And then give it execute privileges with 

```sudo chmod +x /usr/local/bin/docker-compose```

And finally check if the install was a-okay with 

```docker-compose --version```

## Step 3: WordPress
Installing WordPress was actually ridiculously easy, so much so that to this moment I am still unsure if I fulfilled the project requirements fully. The first step is to make a directory for WordPress to house itself in, which is obviously nice and easy with 

```
mkdir ~/wordpress/
cd ~/wordpress/
```

And then the only other thing is to create and edit a file called 'docker-compose.yml' which seems to me like the guide for what the docker container is supposed to do. To create it run 

```sudo nano docker-compose.yml```

And then copy-paste the following into that there file, writing out and saving as well.

```
version: '3.3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

With that done, run 

```docker-compose up -d``` 

To start the container, let it do its thing, and then go to your browser and go to ```localhost:8000``` and it should pop up with a WordPress Admin page, allowing you to create and edit your own page, as shown below.

![Admin Lanugage](https://i.ibb.co/yYtKJmm/docker-admin-language.jpg)
![Admin Setup](https://i.ibb.co/kgMkNLZ/docker-admin-setup.jpg)
![Admin Done](https://i.ibb.co/fk0mHYT/docker-admin-done.jpg)

These are all of the steps I took to install WordPress and navigate to the admin page, and it should be completely reproducible, again, all credit to Hostinger Tutorials.
_______________________________________________________________________________________________________________________________________________________________________________
# Arch Installation and Customization Guide
## Introduction
This guide is for the purpose of going through the step by step process by which I installed and customized my own version of Arch Linux for my System Admin class. It should be robust enough that by following the steps and the steps alone you can completely recreate what I have on my machine. That being said, this is for a project and I am installing Arch on VMWare Workstation. If you are trying to install Arch to boot normally on a machine, this guide is perhaps not best for you. Also, there will be mentions of a" classroom gateway" and the instructors, Codi and Sal, that only pertains to me and my project, so ignore those.

# The ISO File
The first step to installing Arhc Linux is downloading the iso file from any one of various mirrors. Select one from [this website](http://mirrors.acm.wpi.edu/archlinux/iso/2021.10.01/) and download it.

Then, to ensure the file is legitamate, right click it and mouse over Hash CRC and select SHA1. Hash CRC will only be an avaiable option if you have 7-Zip installed, so go ahead and get that if you don't have it. SHA1 Should pull up a window, take a second, and then spit out a string of numbers and letters, check it against 

```
77A20DCD9D838398CEBB2C7C15F46946BDC3855E
```

If it does not match up, try a different mirror.

## Booting from the ISO (VMWare)
If you are making a VM of Arch, go into VMWare and make a new VM, selecting the iso file. It won't recognize what type of OS it is so select 'Linux' and "Other Linux 5.x and alter kernel 64-bit" and name it "ArchLinux" or equivalent. Give it 20GB of Disk Space if you have enough available, and 2 GB (2048 MB) of Memory. Then, finalize the VM itself. 

Before actually starting it, go inot your Documents folder and find the Virtual Machines folder and navigate to ArchLinux, then, open the ArchLinux.vmx file and add

```
firmware="efi"
```

as the second line. Then, just go ahead and power on the VM and hit enter on Arch Medium when it asks for what to do next, it should begin installing.

### Note
You are booting into the iso file itself, there are several things that need to be done before booting into Arch itself, so make sure you do not actually shut down/restart the VM until everything is done, otherwise it might be difficult to rectify it and you might need to restart completely.

Initially I didn't have 7-Zip installed, so i had to get that, then I tried to get a torrent file but didn't realize so the SHA was messed up, then finally got the right file and booted into VMWare. Unfortunately, I had forgotten to add the firmware="efi" line so I had to restart it all anyway.

## Partitioning the Drives
You now should be in Arch Linux, with a command line up on the screen and essentially nothing else. The first thing to do is to separate the main drive into one for the root drive and one for EFI. To do this, take a look at the current drives are with the command

```
fdisk -l
```

It should pull up a disk labeled /dev/sda and likely one labeled /dev/loop0. Completely ignore /dev/loop0 and then enter

```
fdisk /dev/sda
```

to actually focus in on the drive. 

First make a new partition with ```n```, to be the efi partition, then ```p``` to label it primary, then ```1``` to designate it partition 1, then just hit enter for the defualt on the third option, and then ```+512M``` for size. It should then put you back to the focus on sda, where you need to change the type of the partition by entering ```t```, and then ```ef``` for EFI.

Next, make the root partition with ```n``` again, and then just enter, enter, and enter again for all of the default settings, letting it take up the remaining availabe size. Entering ```w``` will write the changes to the drive and kick you back to the command line itself

### Note
It took me ages to actually figure out focusing in on sda0, I kept trying to create the partitions inherently without splitting them off from a single existing one

## Creating Filesystems
Now that we have the different partitions, we need to format the filesystems for each one. The EFI one needs a different type than the root partition so first run 

```
mkfs.fat -F32 /dev/sda1
```

for the EFI partition, and

```
mkfs.ext4 /dev/sda2
```

for the root partition.

## Before Installing Arch
When you actually install Arch, you might find that it is a worryingly slow process. To combat this, you can test different mirrors for their speed/connection before actually installing. 

To do this, first run

```
pacman -Syy
```

to sync the pacman repository (the package manager for Arch) so you can download and install things, then run

```
pacman -S reflector
```

to install the reflector package which will allow you to serach for and test different mirrors. 

Next run

```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

to backup the standard mirrolist for Arch, then finally

```
reflector -c “US” -f 12 -l 12 -n 12 --save /etc/pacman.d/mirrorlist
```

to start testing the diffferent mirrors and finding the good ones.

### Note
I found this method from a different Arch Install Guide on itsfoss.com and it works perfectly everytime. I have tried it the way the wiki entails and it was much slower so I just default to this.

## Installing Arch
You are now ready to actually install arch, remember, you are currently runnning off of the iso file. To install Arch to the root partition, first mount it with

```
mount /dev/sda2 /mnt
```

and then actually install it with

```
pacstrap /mnt base linux linux-firmware
```

It might take a while, but it should install fine.

## Configure the Arch system
Now that Arch Linux is actually installed, run

```
genfstab -U /mnt >> /mnt/etc/fstab
```

to generate an fstab file that defines how everything is mounted into the file system. 

With that, you should be able to enter the mounted disk as root with

```
arch-chroot /mnt
```

### Setting timezone
To find a timezone to set your Arch to, first list them all with

```
timedatectl list-timezones
```

select one that is relevant to you, then run

```
timedatectl set-timezone Region/City
```

where 'Region/City' matches your timezone.

### Setting locale
To do the same with locale, first install the text editor nano with

```
pacman -S nano
```

and edit the locale.gen file with

```
nano /etc/locale.gen
```

and find your desired language/locale, deleting the '#' on its line to uncomment it. Then run

```
locale-gen
```

to actually generate the config file and then run

```
echo LANG=language_COUNTRY.UTF-8 > /etc/locale.conf
```

```
export LANG=language_COUNTRY.UTF-8
```

where 'language_COUNTRY' matches the language/locale you uncommented.


### Note
The first time I tried to set up the locale I didn't realize I actually had to uncomment the line in locale.gen so when I got into Arch with GNOME the terminal wasn't working which was fun

### Network Configuration
Next create a hostname file with your computer's hostname with

```
echo name > /etc/host_name
```

where 'host_name' is whatever name you want to use, I use 'myarch.'

Then create a hosts file with

```
touch /etc/hosts
```

and edit it with

```
nano /etc/hosts
```

so that it reads:
```
127.0.0.1	localhost
::1		localhost
127.0.1.1 host_name
```
Again where 'host_name' is whatever name you chose.

### Set up root password
Now set up the password for the system root with the command

```
passwd
```

## Install Grub bootloader
When you are all done with the iso, you will need a bootloader to boot without it, Grub is a common and effective one. To install the packages, run

```
pacman -S grub efibootmgr
```

and create a directory for it with

```
mkdir /boot/efi
```

and mount it with

```
mount /dev/sda1 /boot/efi
```

then actually install grub with

```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

## Installing GNOME desktop environment
With Grub set up, you can install a desktop environment to able to actually use Arch easily, so this guide will show how to install and configure GNOME, a common and imple desktop environment. 

First, install the packages for Xorg to be a display server with

```
pacman -S xorg
```

just presseing enter will default install all so either do that or be more discerning. 

Then get the GNOME packages with

```
pacman -S gnome
```

with the same default all.

Then actually enable the display maanger GDM for Arch with

```
systemctl enable gdm.service
```

Also, before getting out of the iso file boot, install the packages for the Arch Network Manager with

```
pacman -S networkmanager
```

and then enable it with

```
systemctl enable Networkmanager.service
```

# Booting into Arch itself (without iso file)
Now that everything is configured and nicely set up, run

```
exit
```

to return from the chroot and then

```
shutdown now
```

to end the Arch VM itself. 

Before actually starting it again, edit your Arch VM settings and chang the 'CD/DVD (IDE)' from 'Use iso image file' to 'Use physical drive' and choose 'Auto detect'. Then go ahead and run the VM again.

It should pull up Grub first, asking what you would like to do, defaulting to booting into Arch if you hit enter or let it sit for 5 seconds. Then it should boot in Arch with GNOME as the desktop environment. Since there are no user accounts yet, enter 'root' as the username and then enter the root password you set earlier to get in.

### Note
I have heard of people having trouble with getting the DE to work but honestly this method for getting GNOME that I got from the previously mentioned itsfossed.com was perfect and I am eternally grateful

# Customizing Arch
## Configuring sudo
The first thign to do to make your Arch work properly and work well is to install sudo, the classic root privilige command. To do so , search GNOME's Activities for the Terminal and then run

```
pacman -S sudo
```

and then make a sudo group with

```
groupadd sudo
```

At the moment the group doesn't actually do anything, so edit the /etc/sudoers file with

```
nano /etc/sudoers
```

and remove the # from the line

```
#%sudo  ALL=(ALL) ALL
```

This will set it so that any user in the sudo group can run all commands with sudo.

### Note
The fact that sudo isn't inherent to Arch initially threw me off at first and I tried to get it to work with wheel but got confused and just installed sudo anyway, plus the project declaration says 'sudo' so I got sudo

## Making user accounts
Next, add your own user account to the system with

```
useradd -m -G sudo user_name
```

where 'user_name' is whatever name you choose for the account. Then set a password for that account with

```
passwd user_name
```

Now, add two more user accounts for the instructors, Sal and Codi with

```
useradd -m -G sudo sal
useradd -m -G sudo codi
```

Then run both

```
passwd sal
passwd codi
```

and give both accounts the password 'GraceHopper1906' as a default. This password needs to be changed whenever they first login so set each of those passwords to expired with 

```
passwd -e sal
passwd -e codi
```

to force them to change it.

### Note
I tried to get the useradd commands to set the passwords at the same time as creation with -p and I still don't know why it didn't work, but it wasn't that big of a deal to set them immediately after.

### Fish
With those three accounts created, go ahead and log out of the root account and log back into yours, then open up the terminal again. 

The first thing to do is to install an alternative shell, a more user-friendly one than bash named fish. To install it just run

```
sudo pacman -S fish
```

and either open it from the Activities or run

```
fish
```

in the terminal whenever you'd like. 

## Classroom Gateway with SSH
Next, get into the classroom gateway with ssh, first by installing ssh itself with

```
sudo pacman -S openssh
```

then actually use it to reach cognizant by running

```
ssh -p53997 cog_user@129.244.245.21
```

where 'cog_user' is your cognizant user name. Then, in cognizant, run

```
ssh -p53997 admin@192.168.2.your_ip
```

or

```
ssh -p53997 user@192.168.2.your_ip
```

where your_ip is your individual gateway ip. Just run

```
exit
```

to get back to cognizant and then

```
exit
```

again to get back to the normal terminal.

### Adding Color to Arch
Next, you are going to add some color to the terminal with help from Average Linux User. He has prepared some files to do this easily so run

```
curl https://averagelinuxuser.com/assets/images/posts/2019-01-18-linux-terminal-color/Linux_terminal_color.zip
```

and then extract them with

```
unzip /home/user_name/Downloads/Linux_terminal_color.zip
```

where 'user_name' is still your account name.

Then, before messing with any of the settings files, back them up with

```
cp .bashrc .bashrc.backup
sudo cp /etc/bash.bashrc /etc/bash.bashrc.backup
```

With that squared away, just run

```
sudo mv bash.bashrc /etc/bash.bashrc
sudo mv DIR_COLORS /etc/
mv .bashrc ~/.bashrc
``` 

to move his files into their necessary locations. Run

```
ls
```

to see how that changed things.

Next, to add color to pacman you must edit the pacman.config file so first back it up with

```
sudo cp /etc/pacman.conf /etc/pacman.conf.backup
```

and then

```
sudo sed -l 's/#Color/Color/g' /etc/pacman.conf
```

to actually edit it automatically. Then run

```
sudo pacman -Suy
```

or

```
man pacman
```

to see the difference.

Finally, add some syntax coloring to the nano text editor by changing /etc/nanorc, so back it up first with

```
sudo cp /etc/nanorc /etc/nanorc.backup
```

then edit the file with

```
sudo nano /etc/nanorc
```

and add

```
"include /usr/share/nano/*.nanorc
```

to the end. Then just open the same file back up with

```
sudo nano /etc/nanorc
```

again to see what changed. This will also include syntax coloring for several scripting languages like python.

### Note
The Average Linux User, whose website is averagelinuxuser.com, saved me a ton of time here as he has a guide that details all of this adn made it easier to use. It took me a while to figure out the curl, but now that it works it's pretty slick.

## Auto Boot
Now, make it so that Arch boots immediately (skipping the wait time in Grub) by running

```
sudo nano /etc/default/grub
```

and changing the line that says

```
GRUB_TIMEOUT=5
```

to 

```
GRUB_TIMEOUT=0
```

Then you need to update the config file itself with

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Aliases
Next, add some aliases to your Arch to make certain commands quicker with shortcuts. To do so you need to edit the ~/.bashrc file with

```
sudo nano ~/.bashrc
```

and just put any aliases you want to add under the

```
alias ls='ls -color=auto'
```

that was added when we added color to the terminal.

Some helpful aliases are shown below that you can simply copy into the file or you can add your own or any you find online that seem helpful.
```
alias cd.1=’cd ..’ 			#Navigates to the current directory’s parent
alias cd.2=’cd ../..’			#Navigates to the parent’s parent
alias cd.3=’cd ../../..’			#Navigates to the parent’s parent’s parent
alias update=’sudo pacman -Syu’	#Updates all available packages
alias pac=’sudo pacman -S’ 		#Shortcut to install the package listed after pac
alias c=’clear’				#Clears the terminal of text
alias please=’eval “sudo $(fc -ln -1)”’	#Runs the last command with sudo privileges 
```

After you are done editing that file, run

```
source ~/.bashrc
```

to update everything and then try them out.

If you ever want to delete an alias, just edit that file again, delete the corrosponding line, then run

```
source ~/.bashrc
```

again.

## Gateway Alias
You can also make an alias to get into cognizant much easier by adding the line

```
alias cognizant=’ssh -p53997 cog_user@129.244.245.21’
```

to that same ~/.bashrc file, where 'cog_user' is your cognizant username.

To do the same but for getting into the gateway itself, first get into cognizant, then run

```
nano ~/.bashrc
```

to get into cognizant's version of the same file and add

```
alias gateway-admin=’ssh -p53997 admin@129.168.2.your_ip’
alias gateway-user=’ssh -p53997 user@129.168.2.your_ip’
``` 

to the very end, where your_ip is your individual gateway ip. Then run 

```
source ~/.bashrc
```

to update it.

### Note
I found a bunch of these aliases on reddit and personally the 'please' one is hysterical and legit have used it a lot throughout the process

## Helpful Arch Packages
You should also go ahead and add a video player to arch with

```
sudo pacman -S vlc
```

and a music player with

```
sudo pacman -S cmus
```

Libre office is an open soruce version of Microsoft Office and its entire suite can be installed with

```
sudo pacman -S libreoffice
```

Conky is a neat little app that runs on the desktop and shows live info about the CPU, Memory, Storage, and more and can be installed with

```
sudo pacman -S conky
```

then opened from the Activities.

## GNOME Appearence
Finally, go to Arch Activities and search 'Tweaks' and open it. Tweaks allows you to change some of the finer appearence and function realted options about GNOME, such as desktop background, changing the theme from light mode to dark mode, and even the font of the text of different applications.

# Conclusion
If you followed the guide step by step it sohuld have gotten a clean install of arch, then customized it and added some quaility of life features that would be helpful for any new or old Linux user. Thank you. 

-Benjamin 'Ben' Hughes
