# Arch Installation and Customization Guide
## Introduction
This guide is for the purpose of going through the step by step process by which I installed and customized my own version of Arch Linux for my System Admin class. It should be robust enough that by following the steps and the steps alone you can completely recreate what I have on my machine. That being said, this is for a project and I am installing Arch on VMWare Workstation. If you are trying to install Arch to boot normally on a machine, this guide is perhaps not best for you. Also, there will be mentions of a" classroom gateway" that only pertains to me, my classmates, and my professors, ignore it otherwise.

## The ISO File
The first step to installing Arhc Linux is downloading the iso file from any one of various mirrors. Select one from [this website](http://mirrors.acm.wpi.edu/archlinux/iso/2021.10.01/) and download it.

Then, to ensure the file is legitamate, right click it and mouse over Hash CRC and select SHA1. Hash CRC will only be an avaiable option if you have 7-Zip installed, so go ahead and get that if you don't have it. SHA1 Should pull up a window, take a second, and then spit out a string of numbers and letters, check it against ```77A20DCD9D838398CEBB2C7C15F46946BDC3855E```. If it does not match up, try a different mirror.

## Booting from the ISO (VMWare)
If you are making a VM of Arch, go into VMWare and make a new VM, selecting the iso file. It won't recognize what type of OS it is so select "Other Linux 5.x and alter kernel 64-bit" and name it "ArchLinux" or equivalent. Give it 20GB of Disk Space if you have enough available, and 2 GB (2048 MB) of Memory. Then, finalize the VM itself. 

Before actually starting it, go inot your Documents folder and find the Virtual Machines folder and navigate to ArchLinux, then, open the ArchLinux.vmx file and add ```firmware="efi"``` as the second line. Then, just go ahead and power on the VM and hit enter on Arch Medium when it asks for what to do next, it should begin installing.

### Note
You are booting into the iso file itself, there are several things that need to be done before booting into Arch itself, so make sure you do not actually shut down/restart the VM until everything is done, otherwise it might be difficult to rectify it and you might need to restart completely.

## Partitioning the Drives
You now should be in Arch Linux, with a command line up on the screen and essentially nothing else. The first thing to do is to separate the main drive into one for the root drive and one for EFI. To do this, take a look at the current drives are with the command ```fdisk -l```. It should pull up a disk labeled /dev/sda and likely one labeled /dev/loop0. Completely ignore /dev/loop0 and then enter ```fdisk /dev/sda``` to actually focus in on the drive. 

First make a new partition with ```n```, to be the efi partition, then ```p``` to label it primary, then ```1``` to designate it partition 1, then just hit enter for the defualt on the third option, and then ```+512M``` for size. It should then put you back to the focus on sda, where you need to change the type of the partition by entering ```t```, and then ```ef``` for EFI.

Next, make the root partition with ```n``` again, and then just enter, enter, and enter again for all of the default settings, letting it take up the remaining availabe size. Entering ```w``` will write the changes to the drive and kick you back to the command line itself

## Creating Filesystems
Now that we have the different partitions, we need to format the filesystems for each one. The EFI one needs a different type than the root partition so first run  ```mkfs.fat -F32 /dev/sda1``` for the EFI partition, and ```mkfs.ext4 /dev/sda2``` for the root partition.

## Before Installing Arch
When you actually install Arch, you might find that it is a worryingly slow process. To combat this, you can test different mirrors for their speed/connection before actually installing. 

To do this, first run ```pacman -Syy``` to sync the pacman repository (the package manager for Arch) so you can download and install things, then run ```pacman -S reflector``` to install the reflector package which will allow you to serach for and test different mirrors. 

Next run ```cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak``` to backup the standard mirrolist for Arch, then finally ```reflector -c “US” -f 12 -l 12 -n 12 --save /etc/pacman.d/mirrorlist``` to start testing the diffferent mirrors and finding the good ones.

## Installing Arch
You are now ready to actually install arch, remember, you are currently runnning off of the iso file. To install Arch to the root partition, first mount it with ```mount /dev/sda2 /mnt``` and then actually install it with ```pacstrap /mnt base linux linux-firmware```. It might take a while, but it should install fine.

## Configure the Arch system
Now that Arch Linux is actually installed, run ```genfstab -U /mnt >> /mnt/etc/fstab``` to generate an fstab file that defines how everything is mounted into the file system. 

With that, you should be able to enter the mounted disk as root with ```arch-chroot /mnt```.

### Setting timezone
To find a timezone to set your Arch to, first list them all with ```timedatectl list-timezones```, select one that is relevant to you, then run ```timedatectl set-timezone Region/City```. Where 'Region/City' matches your timezone.

### Setting locale
To do the same with locale, first install the text editor nano with ```pacman -S nano``` and edit the locale.gen file with ```nano /etc/locale.gen``` and find your desired language/locale, deleting the '#' on its line to uncomment it. Then run ```locale-gen``` to actually generate the config file and then run ```echo LANG=language_COUNTRY.UTF-8 > /etc/locale.conf``` and ```export LANG=language_COUNTRY.UTF-8``` where 'language_COUNTRY' matches the language/locale you uncommented.

### Network Configuration
Next create a hostname file with your computer's hostname with ```echo name > /etc/host_name``` where 'host_name' is whatever name you want to use, I use 'myarch.'

Then create a hosts file with ```touch /etc/hosts``` and edit it with ```nano /etc/hosts``` so that it reads:
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myarch
```

### Set up root password
Now set up the password for the system root with the command ```passwd```.

## Install Grub bootloader
When you are all done with the iso, you will need a bootloader to boot without it, Grub is a common and effective one. To install the packages, run ```pacman -S grub efibootmgr``` and create a directory for it with ```mkdir /boot/efi``` and mount it with ```mount /dev/sda1 /boot/efi```, then actually install grub with ```grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi and grub-mkconfig -o /boot/grub/grub.cfg```.

## Installing GNOME desktop environment
With Grub set up, you can install a desktop environment to able to actually use Arch easily, so this guide will show how to install and configure GNOME, a common and imple desktop environment. 

First, install the packages for Xorg to be a display server with ```pacman -S xorg```, just presseing enter will default install all so either do that or be more discerning. 

Then get the GNOME packages with ```pacman -S gnome``` with the same default all.

Then actually enable the display maanger GDM for Arch with ```systemctl enable gdm.service```.

Also, before getting out of the iso file boot, install the packages for the Arch Network Manager with ```pacman -S networkmanager``` and then enable it with ```systemctl enable Networkmanager.service```

## Booting into Arch itself (without iso file)
Now that everything is configured and nicely set up, run ```exit``` to return from the chroot and then ```shutdown now``` to end the Arch VM itself. 

Before actually starting it again, edit your Arch VM settings and chang the 'CD/DVD (IDE)' from 'Use iso image file' to 'Use physical drive' and choose 'Auto detect'. Then go ahead and run the VM again.

It should pull up Grub first, asking what you would like to do, defaulting to booting into Arch if you hit enter or let it sit for 5 seconds. Then it should boot in Arch with GNOME as the desktop environment. Since there are no suer accounts yet, it will ask you for the root password to get in.

### Costomizing Arch




You can use the [editor on GitHub](https://github.com/hughesbenm/hughesbenm.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/hughesbenm/hughesbenm.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
