# Linux Mint Blog/ Manual
Here's an ancient blog entry of what I did when I first setup my Linux Mint desktop machine with a Cinnamon UI that I would spend the rest of my internet career with.  

###### TOC:

- [Quick Install](#quick-install)
- Customizations
- Bugs
- Hotkeys
- Browsers
- System Stuff


- Social
- Backups
- Anonymity
- Hacking


### Quick Install

Here's a no fluff procedure to bring the system up to my own personal specifications.  

```
    setpci -s 00:02.0 F4.B=19  # set brightness to 19% to save batteries and eyes.  
    
    sudo apt-get install xbindkeys xbindkeys-config xautomation nbtscan grdesktop xcalib ekiga vim xclip dconf-tools tree youtube-dl sshfs openssh-server;
    sudo apt-get install docky keepassx virtualbox treeline;
    sudo apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf mysql-server mysql-client libmysqlclient15-dev nodejs
    
    \curl -sSL https://get.rvm.io | bash -s stable --ruby
    # do rvm implode to fully uninstall .rvm/ then run the bash script above again to do a 'fresh install'
```

You might want to fix up your database
```
    $  mysql -u root -p
    CREATE USER 'user'@'localhost' IDENTIFIED BY 'pass';
    GRANT ALL ON database.* TO user@localhost IDENTIFIED BY 'pass';
    GRANT ALL ON database.* TO user@localhost IDENTIFIED BY 'pass';
    GRANT ALL ON database.* TO user@localhost IDENTIFIED BY 'pass';
```


Here's a list of hidden files and directories off of the home directory that are worth transfering to fresh installs.  
```
    #.cache/docky/docky.desktop.en_US.UTF-8.cache
    #~/.local/share/docky/plugins
    .gconf/apps/gnome-terminal
    .config/chromium
    .config/google-chrome
    .gconf/apps/docky-2
    
    .eclipse      # a 'just ok' IDE
    .gnupg        # GnuPG keys used by enigmail in thunderbird
    .mozilla      # fire fox
    .ssh          # RSA keys for authenticating with other linux machines over SSH
    .shortcuts    # Where I personally store all my shortcuts to drop into docky
    .thunderbird  # Email client
    .vim          # The most fun text editor
    .xchat2       # Xchat IRC client
    .VirtualBox   # A program that lets you run Windows XP while linuxing
    
    .bash_login
    .bashrc
    .my_aliases
    .profile
    .gitconfig
    .xbindkeysrc
    
    # Other good things to backup
    /etc/fstab
    /etc/samba/smb.conf
    
    .gas
    .config/hub
```


###Customizations

::Fix alt-tab timeout (Mint 14.1)

/usr/share/cinnamon/js/ui/altTab.js
```
    const POPUP_DELAY_TIMEOUT = 0; // milliseconds
```



::Fix menu search to be more 'command line' styled (blind searching is ok)
This tweak allows the use of the main menu's search bar as though it were a command line.  E.G. if one wanted to open up their trusty irc client, they would need to type 
1)  [windows key] 
2)  type 'irc'
3)  Wait 150ms
4)  Hit enter when they see "XChat IRC" become the first member of the list.  

If we reduce the Mainloop timeout interval to 20ms, the procedure becomes
1)  hit windows key
2)  type 'irc'
3)  Hit enter as that 20ms will have already gone by since last typing 'c'  (special note:  it's extremely difficult to hit the enter key sooner than 20ms after having your finger at home row position, but if you type faster than me, then reduce the loop to something like 10ms instead.)

/usr/share/cinnamon/applets/menu@cinnamon.org/applet.js
```
    this._searchTimeoutId = Mainloop.timeout_add(150, Lang.bind(this, this._doSearch));
    // That should be...
    this._searchTimeoutId = Mainloop.timeout_add(20, Lang.bind(this, this._doSearch));
```

In the future, it might be better to have the 'enter key' press perform it's own '_doSearch' and choose the top search as long as the arrow keys haven't been used to move the focus since typing the last letter (that's pretty ugly to impliment though, i gotta agree).  


::Detatch screen lockout from closing the lid
When you close the lid, it is implied that you want the screen off and out of your way.  But this doesn't necessarily mean you want to lock down the session to prevent intrusion.  Whereas, when a certain amount of idle time has passed, you may want the screen locked if they consider their laptop to contain private or safegaurded materials.  

$  dconf-editor

org.gnome.desktop.screensaver idle-activation-enabled
org.gnome.desktop.screensaver lock-enabled

ref:  http://forums.linuxmint.com/viewtopic.php?f=208&p=658187


:: Fix udev rules so members of the dialout group can write to things they're meant to

(/etc/udev/rules.d/30-allow_trinket.rules)
```
    SUBSYSTEM=="usb", ATTRS{idVendor}=="1781", ATTRS{idProduct}=="0c9f", GROUP="dialout", MODE="0664"
```


###Bugs

Well... there are a lot of computer manufacturers in the world.  Groups like dell, lenovo, acer-msi-asus, alienware, ibuypower, etc...  And what operating systems do you think these people test their hardware against?  Yep, windows.  So each time an disreputable company like, say, Gateway, comes out with a new laptop, they're releasing a solid piece of hardware if you're using windows XP on it for instance.  However, they're releasing a dumpy alpha release from the stand point of serious linux users.  

::Brightness Control doesn't work in linux mint::
To be fair to gateway, everything else works other than the screen brightness control (the most important control).  I believe the problem has more to do with the issues that intel has with integrated graphics cards than it does to do with Gateway...  

-Problem
Anyway, the problem was that the brightness softkeys change a file in the /proc virtual directory... but to get the actual brightness levels changed, you need to change a special intel file in the /proc directory instead.  I lack the savvy to make the softkeys work as intended, but...


-Solution
The solution for me was to manually set the brightness to my desired value on the terminal (as root!).  

(set the screen brightness to 19, or 19%)
```
    $  su
    <Input Pass>
    $  setpci -s 00:02.0 F4.B=19
```


::Sound sux on Alienware PCs
The solution is in this thread...
http://forums.linuxmint.com/viewtopic.php?f=49&t=122412


::Graphics problems with AMD Radeon HD 7970M Graphics::
The AMD 7970M GPU has a feature they named "Enduro" which means it can automatically switch between the discrete, high power GPU and the on-CPU GPU to save power when there is no demand for GPU calculations.  Unfortunately...  AMD is bad at stuff and it's really buggy in linux.  It won't boot into Ubuntu 12.10 I think... nor Linux Mint 14.1.  The problem might have had something to do with xOrg, but I never really got to the bottom of it.  The solution is to...  Well on some laptops there's a button called I/D GFX that allows you to manually disable Endurio and force just the Intel CPU GPU to be used.  When that's done, you'll be able to install Linux on your laptop (if you have this magic button!  Some mobos don't have a keyboard button or a bios option at all!).  Once you've installed Linux you can then install "legacy" drivers and then switch to what ever GPU you want to use for linux and things should be fine (for the most part...)

http://forums.linuxmint.com/viewtopic.php?f=47&t=130254&sid=93dd3308734c16f47cb9dc6748b1e404



###Hotkeys
Go to Software Manager and install xbindkeys.  Then install xbindkeys-config.  launch xbindkeys-config from a terminal window and you should be good to go for configuring the following hotkeys.  

You'll also need to install 'xautomation' to "send keystorkes" which is helpful for, say, turning your capslock button into an escape key if you aren't using your capslock key very much.  

recap/ Code behind:
```
    $  apt-get install xbindkeys xbindkeys-config xautomation
    
    $  xbindkeys-config
```


::Hotkeys you should install OR have by default.  

Desktop
	Win+d          Show desktop (default)
	Ctrl+Alt+t          Spawn Terminal (default)
	Ctrl+Alt+backspace  Kill the windows manager if it freezes (default)

	Ctrl+Alt+e        Spawn file explorer
	

Web Browsers and Editors
	mouse button 6 (scroll left)    =>     Control+Shift+Tab    (move tab left in )
	mouse button 7 (scroll right)    =>     Control+Tab    (move tab right)

recap/ Code behind:
(~/.xkeybindsrc)
```
###########################
# xbindkeys configuration #
#keystate_numlock = enable
#keystate_scrolllock = enable
#keystate_capslock = enable
###########################

#Show/Hide desktop
"if xprop -root  _NET_SHOWING_DESKTOP|egrep '= 1' ; then  wmctrl -k off ; else wmctrl -k on ; fi"
    Control+Alt + d

#FireFox
"firefox"
    m:0x15 + c:25
    Control+Shift + w

#nemo
"nemo -n ~/"
    m:0x1c + c:26
    Control+Alt + e

#Invert Screen Colors
"xcalib -i -a"
    Control+Alt + m

#Tab Left
"xte 'keydown Control_L' 'keydown Shift_L' 'key Tab' 'keyup Shift_L' 'keyup Control_L'"
   b:6

#Tab Right
"xte 'keydown Control_L' 'key Tab' 'keyup Control_L'"
   b:7

#
# End of xbindkeys configuration
```


ref:  http://www.linuxquestions.org/questions/linux-software-2/how-to-show-desktop-in-xfce4-601161/

ref:  http://askubuntu.com/questions/85850/how-to-remap-a-key-combination-to-a-single-key



::Remap the windows key::

I'm starting to get used to ctrl+alt+d rather than Super+d, so I'm personally not using this trick...

But a trick, if you want to use xbindkeys to bind, say, windows key + e, then do..

#FireFox
"firefox"
    m:0x50 + c:26
    Mod2+Mod4 + e

...then you can make linux register the key combo by first opening the start menu, and then pressing the key combo...  You could also change the way the windows key works by editing the Keyboard Layout -> Layouts -> Options  to make the windows key be a hyperkey.  


disable windows key from messing up
ref:  http://superuser.com/questions/433724/how-do-i-disable-the-keyboard-shortcut-for-menu-in-linux-mint-13



::Create Keybindings with built in GUI for Linux Mint::

You could also just use the built in feature, but I don't like it because it's unclear where it's dotfiles are stored which complicates migrations.  Hit windows key, and type 'keyboard'.  Then press enter.  You should now be in the Keyboard setting panel.  Click the shortcuts tab.  This panel allows you to create custom shortcuts.  It may also be useful to disable default hotkeys in here, if you'd like to do everything in xbindkeys, but that too makes LinuxMint to LinuxMint migrations more complicated.  





###Browsers

-Ctrl+l
Just a little tip for using web browsers, when you want to highlight the search bar, DON'T use the mouse.  Linux is keyboard oriented, so they make highlighting with the mouse more difficult I think.  Instead type ctrl+l and that will highlight the address bar.  This is more efficient because the next thing you will likely do is ctrl+c to copy, or just typing in a search query/url and so your hands will already be at the keyboard.  

-Ctrl+Shift+y
Open downloads window.  


::Setup FireFox

:Searching
First thing's first, setup FireFox so you can browse.  Firefox is installed by default, so Linux Mint get a +1 here.  However, Google, the largest search engine on the web isn't sharing ANY of it's revenue with Linux Mint which I find quite contemptable.  But because I'm a programmer and a serious web searcher, I have no choice to add google to the search engine bar an bind them to the key 'g'...  Perhaps other search engines will become better at searching API documentation and I will be able to switch to them.  Or perhaps google will share it's wealth with what seems to be the greatest OS in the world.  I suggest using one of the partner search engines to look for how to add the google search engine to FireFox in mint...  as a practice, to see if it works well.  

Don't foget to change the default search engin by going to 'about:config' (hint: keyword.url)

:Mouse Gestures
You need mouse gestures immediately!  Go to https://addons.mozilla.org/en-US/firefox/addon/all-in-one-gestures/

I change... 
open new tab:  D
Copy tab:  DU
Close tab:  DR

:Tab Mix Plus
You need to get tab mix plus in order to have the tabs open up in a convienient fashion.  


::Setup Chrome

Everyone knows you need more than one web browser to get a good experience on the web.  That means you're going to go to google and downloading their web browser.  Chrome is the browser you'll use for just quickly getting to a site you want.  You want have your tabs save when you close chrome, that's what Firefox is for.  You'll be downloading a .deb file.  Simply double click this .deb file and it will walk you through the packages installation.  

Once the installation is complete, you can get to chrome by hitting the windows key (I know, ironic, isn't it) and typing in 'chrome'.  





###System Stuff


::Dual Booting::

I expected dual booting to be a very lengthy technical section, but you don't need to follow a lengthy tutorial, in fact, I just "winged it" a few seconds ago and everything worked out well (for me, caveat emptor).   

First, windows uses a method called (aparently...) MBR, where bits of data are written to a very particular place on the HDD.  This alows the bios to boot up into windows properly.  Linux uses something called VBR or something, which I think must be a similar concept according to what I read (lost the links, sry, I have the feeling it got some of the facts wrong anyway...).  The program for linux booting is called grub2, and it sort of stores lists of OSes that can be booted.  Right now, my grub contains an entry for Ubuntu, linux Mint, and also Windows 7.  I now boot into windows from grub instead of using the windows boot loader (MBR).  

So let's say you just finished installing windows 7 on your laptop, and now, to your demise, you're noticing that you can no longer boot into your trusty Linux Mint installation...  Well, you can put in the LM installation CD and run the command 

```
    #  This command mounts the partion you'd like 
    #  to reinstall grub on.  
    $  sudo mount /dev/sdXY /mnt
    
    #  This is the grub installation command
    $  sudo grub-install --root-directory=/mnt/ /dev/sdX
    
    
    #  this tells grub to look over all the HDDs and 
    #  finds Operating Systems and adds them to the boot menu.  
    $  sudo update-grub 
```

...where sdXY is the ext4 partion (or w/e format you used) for the root file system of LM, and sdX is the drive on which said partition exists.  Check teh reference if you don't catch what I'm saying.   

ref:  http://forums.linuxmint.com/viewtopic.php?f=42&t=58174&hilit=grub2



::Grub Customizer::

Sometimes really messed up stuff happens with the grub boot loader and you wind up with options and duplicates that you really don't want.  Consult this to install a GUI to clean up the boot options.  Grubs cfg text file is complete foobar, it's not advisable to edit it manually.  

http://forums.linuxmint.com/viewtopic.php?t=84055


::NaUtilus::
Nautilus is like windows explorer.  

Related Shortcuts
	Ctrl + L      Show address bar
	Ctrl + H      Show/hide hidden files
  Ctrl + Space  Preview media file


::Create Desktop Shortcuts::

Let me just show you the template of an existing desktop shortcut.  Create them with vi or nano.  

(~/Desktop/Aptana.desktop)
```
    #!/usr/bin/env xdg-open
    [Desktop Entry]
    Type=Application
    Name=Aptana
    Exec=/home/kentos/bin/Aptana_Studio_3/AptanaStudio3
    Icon=/home/kentos/bin/Aptana_Studio_3/icon.xpm
```


::Create a Linux Mint Start Menu Shortcut::

Right click the menu and choose "Edit menu" to add/remove items.  


::XBMC::

The standard 'Totem' mediaplayer is a fully functional media player that windows would be blessed with if they installed on their systems by default.  But it doesn't look completely rad.  On the other hand, XBMC does, so let's install that.  

```
    $  apt install xbmc
```

To use XBMC, you need to log completely out and then when you loging, click the gear icon to login with the XBMC session.  This is like replacing your xbox dashboard but for linux, and you aren't replacing it for ever, just until you log back in normally.  



::Ping a windows computer by name (netbios name)::

There are two approaches to 'pinging' windows machines from Linux Mint.  The first approach is to restructure the way your computer operates on the network in a very sloppy and clumsy way.  The second approach does things without using ping at all and instead using nbtscan to resolve netbios names and see if hosts are alive.  

1)  
Here is a novice approach to turning on 'pinging' for your linux machine, but following this is strongly discouraged because it degrades network performance.  I'm only showing this here because it appears on other parts of the Internet and that should be acknowledged.  Yes, it works in a hurry, but even your web browsing can be slowed down with this approach, so please skip to approach 2.  

    Install dependencies 
```
    $  apt-get install winbind
```

(vi /etc/nsswitch.conf)
```
    hosts:        ...bla blah... wins     # insert the word "wins"
```

ref:  https://www.zulius.com/how-to/resolve-windows-netbios-names-from-linux/


2)
Instead of using ping to resolve netbios names to IP addresses or check if computers are online, you can run this command.  

```
    $  apt install nbtscan
```

```
    $  nbtscan 192.168.1.1/24
    Doing NBT name scan for addresses from 192.168.1.1/24
    
    IP address       NetBIOS Name     Server    User             MAC address      
    ------------------------------------------------------------------------------
    192.168.1.0	Sendto failed: Permission denied
    192.168.1.4      INNER            <server>  <unknown>        00:xx:xx:xx:xx:xx
    192.168.1.5      E5               <server>  <unknown>        00:xx:xx:xx:xx:xx
    192.168.1.11     RE               <server>  <unknown>        00:xx:xx:xx:xx:xx
    192.168.1.10     SAMBA            <server>  SAMBA            00:00:00:00:00:00
    192.168.1.34     KENTOS-NV54-SER  <server>  KENTOS-NV54-SER  00:00:00:00:00:00
    192.168.1.45     PR               <server>  <unknown>        00:xx:xx:xx:xx:xx
    192.168.1.110    AIR              <server>  <unknown>        00:xx:xx:xx:xx:xx
    192.168.1.255	Sendto failed: Permission denied
```

This command is many times more powerful than ping.  Not only is it telling you whether the computers are online or not, it also gives you their IP address AND their macs.  

3)
Yes, there's actually a 3rd way.  You can setup a local DNS server which will allow you to do pings.  I don't know how to do this yet, but eventually I will.  



::Enable pinging windows machines in an alternative way...

http://forums.linuxmint.com/viewtopic.php?f=157&t=108348#p634500


```
    $  sudo dpkg-reconfigure resolvconf         # Answer YES to the first question, NO to the second,  then OK
    $  sudo dhclient -v
```

::Connect to a windows file share::

Ctrl+Alt+e (if you made that hotkey from above).  Then Ctrl+l.  Then type:
smb://Windows_box_name/

Then just type in the password when prompted.  



::Connect to a windows samba share the best way...

To make it so your system can broadcast (and listen to other samba server's broadcasts) add the below uncomented line to the samba config file.  

(/etc/samba/smb.conf)
```
    ;   name resolve order = lmhosts host wins bcast
    name resolve order = bcast host lmhosts wins
```


```
$  sudo service smbd restart
```


::Connect to a printer on a windows computer

http://www.youtube.com/watch?v=nfS_-X8l5Fg

To add a printer to my fresh linux mint, I first typed "printer" in the start menu and opened up the cinnamon printer dialog.  I tried to connect to my networked printer, but I was unable to.  I did see this message, but didn't really know what to make of it.  

[quote]
FirewallD is not running. Network printer detection needs services mdns, ipp, ipp-client and samba-client enabled on firewall.
[/quote]

Then I went through my Linux Mint notes and found a link to a youtube vid (http://www.youtube.com/watch?v=nfS_-X8l5Fg) which mentioned the belown command.  

```
$  system-config-printer
```

I was able to add my networked printer on a windows machine by following along with this wizard.  

So I recommend a solution such as a link to cyctem-config-printer be placed within the cinnamon "printers" dialog menu, or something be done to fix that menu in the first place.  Right now adding networked printers isn't quite possible with out the user googling for the command system-config-printer, and it should be simple enough not to need to reference other materials.  



::Connect to a windows computer via remote desktop::

Windows will give you a little utility called RemoteDesktop, IF you pay for the professional edition of their software.  Well, you can connect to this expensive piece of software from linux too.  A quick search of Software Manager yielded two packages related to 'rdesktop'.  

rdesktop would seem to be the underlying code base which allows you to programmatically interface with a windows computer via the remote desktop protocol.  
grdesktop would appear to be the GUI for rdesktop in gnome.  It depends on rdesktop (and of course, automatically installs it as needed).  

To install the software, run the command...
```
    $  apt install grdesktop
```

Then load it up from the start menu (you can type 'rdesktop' to call it up).  You can go ahead and plug in your windows computer's name and click execute to connect.  If you're unable to connect, that's probably because you need to do 'apt install winbind' and setup /etc/nsswitch.conf properly.  See the section on Ping a windows computer for my help.  



::SSH - Connect to a Linux machine remotely

The client SSH program, open ssh is installed by default on fresh Linux Mint installs, but not the server.  SSH allows you to run commands at the command line... from a different computer!  Cool stuff, plus it's safely encrypted so it's impossible to write a sniffer for the SSH protocol unless you have access to alien computing resources.  Aliens know everything you do, always.  

You can connect to your computer via SSH from a windows computer using something called Putty.exe, and if you're already on linux and you'd like to connect to another Linux machine, you can use open-ssh (installed by default).  I'll list off some example usages of ssh.  

This will attempt an ssh connection to another local machine on your LAN (probably your router; possibly no machine at all is there).  
```
$  ssh root@192.168.1.1
Password:  <insert remote machine's password>
```

That's easy and straight forward, the ssh command specifies first the user to Login as (root in our case) and which server to connect to (192.168.1.1).  And of course you are authenticated by your password.  

We can save that IP address and login name in a special file called ~/.ssh/config

(~/.ssh/config)
```
Host my_router 192.168.1.1
HostName 192.168.1.1
User root
```

Now at the console you can list off the hosts you have saved:
```
$  ssh <press tab twice>
::1                 ff02::2             ip6-loopback
192.168.0.11        ip6-allnodes        ip6-mcastprefix
fe00::0             ip6-allrouters      kentos-NV54-Series
ff00::0             ip6-localhost       localhost
ff02::1             ip6-localnet        my_router
```

It will also autocomplete, so start typing the name and then hit <tab> and it will fill in for you.  You'll notice there's a lot of junk in there... I'm afraid I don't know of a way to clean it up all pretty like, but it's rare that I would need to pull up a host list through ssh instead of just cat`ing out the ~/.ssh/config file.  


You're probably wondering how you can save the password too, aren't you?  Well, passwords are kind of out of style.  Nobody really liked typing them and they aren't ever long enough to truly protect your machine.  So we'll use public/private keys to authenticate instead of passwords.  

Create a public/private key pair
```
$  ssh-keygen -t rsa -b 2048
```


Once you've created a keypair, you can remotely push it to the server you'd like to login to with it. 
```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip
# supply your password for the server
```

::SSH - Enable the Server on Linux Mint

WARNING:  Following these steps requires that you have a strong password for ALL users on your Linux Mint computer!  

To my utter surprise, sshd is not installed on your Linux Mint machine by default.  I guess that's because Linux Mint is designed to be a desktop, so the emphasis is on client applications.  So let's install the ssh server just because it's likely to come in handy for me (I manage multiple linux servers as a part of my job).  

After flipping through the Software Manager, I found the necessary package
```
$  apt install openssh-server
[/cod]


Now you can test the connection by doing...

```
$  ssh root@localhost
Password:  
Welcome to Linux Mint 13 Maya (GNU/Linux 3.2.0-23-generic x86_64)

Welcome to Linux Mint
 * Documentation:  http://www.linuxmint.com
Last login: Thu Sep 13 10:31:20 2012
computer-name ~ # 
```

Yikes, as root?  That's a little scary right?  I mean, even with an 8 character password, it can be brute forced in less than a year if your system becomes under scrutiny.  Let's disable that...

(/etc/ssh/sshd_config)
```
.
.
.
PermitRootLogin no  # Change this from yes to no
.
.
.
```


Restart sshd to allow your configuration changes to take effect.  
```
$  restart ssh
```


That improves your chances well; now the attacker needs to guess your login name and your strong password.  Not too shabby.  


Additionally, you could limit connections to use only public/private keys.  This security is optimal, but for me on my laptop, it's a little too optimal.  

(/etc/ssh/sshd_config)
```
.
.
.
PasswordAuthentication no
.
.
.
```


::Remote Desktop server for linux::

You can actually connect to your linux machine from any windows machine using it's built in Remote Desktop Client.  I played around with this and it seems that you need to manually reset your desktop background to a black screen or it goes painfully slow.  I don't see there being too much of a need for this since windows isn't the ideal desktop OS (thanks again Linux Mint team!) but I figured I'd list this off here just in case you have a linux machine somewhere else, and you'd like to start it jdownloading some file or something so its ready when you arrive (you can't do that from the console that I know of).  

```
$  apt install xrdp
```

I'm removing it from my comp though, I don't see the need to have it on here, and if I do need it, i'll just install it over SSH and then connect.  

::Remote Desktop - Connect to a linux machine remotely
UNFINISHED

You have the option to install Remmina, the remote desktop client for linux if you would like to connect to a linux machine remotely.  Everything is already automatically setup for you so just go to your software manager or run the code below to install the client on your machine.  

```
$  apt install remmina
```

I haven't tested this yet!!


::Startup files::

```
~/.xprofile
~/.bashrc
~/.profile
~/.bash_login

/etc/xdg/autostart/
/etc/network/if-pre-up.d/wireless-tools
/etc/crontab   # set via `crontab -e`

/etc/init # all scripts in here are loaded http://ubuntuforums.org/showthread.php?t=1786173
/etc/network/if-up.d   # All scripts in this directory are run when your network comes online
```

When your linux session first starts up, it finds ~/.xprofile an executes it's contents.  It's called x profile, because it's the profile for X, which is the linux GUI base code.  You may find this knowlege useful.  There's also scripts that get executed when you first login to a bash shell.  ~/.profile and ~/.bashrc and ~/.bash_login are some of these files.  Also note /etc/profile is a system wide version of these files.  






::Putty Style Terminal for Linux Mint::

Usage
	Copy Text:          Simply highlight some text
	Paste Text:         Middle click

The terminal you get when you press Ctrl+alt + T is called "gnome-terminal".  It's annoying to have to copy and paste things by using the context menu, and since ctrl+c is bound to "Close Process" in terminals, you would hope that highlighting any text within the terminal would automatically put it into your copy buffer.  Well, it does that!  But it adds it to it's own special copy buffer, so when you try to right click and paste, it won't paste what you expect.  Instead you have to use middle click to paste.  

ref:  sudo apt-get install parcellite



::Gnome-Terminal::

Shortcuts
  Ctrl+a        Go to start of command line
  Ctrl+e        Go to end of line
  Ctrl+u        Clear command line
  Ctrl+w        Delete Word
  Ctrl+Shift+t  Spawn new tab


CREATE THESE   edit -> Keyboard Settings -> tabs
  Ctrl+Shift + h   tab previous
  Ctrl_Shift + l   tab next





::A Windows 7/ OSX Style Dock Panel (Docky)

Ok, before I start, let me just say, these panels are truly more efficient if you use your computer on the job (programming, IT, bla, blah).  If you work a computer related profession, you likely use suites of applications, and the Dock panel system facilitates this style.  Before windows 7, I was an avid "quick list" user, which in windows allows you to arrange shortcut icons right on the task bar.  I found it hard to switch to the dock panel, but at this point, I agree that ist is far more efficient.  

Go to Software Manager and search for "Docky".  Then just install it.  Use the window key to search for docky to load it up manually.  It should be setup to automatically load up when the user logs in from now on.  Set docky to 'Panel Mode' in the settings to make it most similar to Windows 7.  

I had trouble getting the highlighted overlay to indicate which app was currently in focus.  The fix for me was to download ambience-dark theme and use that.  http://gnome-look.org/content/show.php/Ambiance+Docky+Themes+%28Light+%26+Dark%29?content=125005

You might want to go into your Cinnamon Settings and move the old task bar to the top and set it to auto hide.  You unfortunately don't want to remove it completely because the time is on there, so is the network display and all that luggage.  

```
$  apt-get install docky
```






::>>IDEs<<::

::Aptana::
The best way to install aptana for Linux Mint is by browsing to apanata's website and downloading the linux version.  Then unzip the contents of the download to a folder that's out of the way, but not too far out of the way... (I use ~/bin to store these kinds of files).  

When you're done extracting the file, you need to run Aptana one time as administrator.  After that it will just work.  

(what I just said above, but in shell commands)
```
$  mkdir ~/bin; cd ~/bin
$  wget http://download.aptana.com/studio3/standalone/3.2.2/linux/Aptana_Studio_3_Setup_Linux_x86_64_3.2.2.zip

$  unzip Aptana_Studio_3_Setup_Linux_x86_64_3.2.2.zip
$  rm Aptana_Studio_3_Setup_Linux_x86_64_3.2.2.zip  # we don't need the zip anymore
```



::Light weight Editor::

I was having trouble finding a good notepad style text editor.  My problem was that, in the windows and mac world, you through out the built in editor because it's buggy rubbish.  But that isn't how the Linux world works.  After closer consideration, I've reached an understanding that gEdit is quite good, although I also do a lot of vim work since I'm mostly on the terminal anyway.  I didn't like gedit at first because after I search for some text, I have a habit of hitting escape to clear the search box from the screen... but pressing esc after searching text in gedit is a way of undoing the search --slamming your view back to where you came from.  This was very jarring for me, especially when I didn't even know where the view was switching to.  Below is a crash course on gEdit, listing off all the feature settings and usages that I find useful.  

::gEdit
Ctrl+f       search
    tab      end search
    esc      undo search
    Ctrl+g   Find next


configuration
Edit -> preferences -> View (tab)
  Display line numbers (don't worry, we'll change it so it's not ugly)
	Highlight matching brackets
	
Edit -> preferences -> Editor (tab)
	Insert spaces instead of tabs
	Enable automatic indentation

Edit -> preferences -> Font & Colors (tab)
	Color Scheme:  Cobalt or Oblivion

Edit -> preferences -> Plugins (tab)
    Change Case

Tools -> Highlight Misspelled Words


Those are all the cool features.  Of course there's plugins you can download too if you'd like more features!  I plan to look into it more when I have some free time.  I'd love to have a button in this editor that just runs the script and displays it's outputs in a small console window.  

Follow up on:
  http://rbjl.net/58-rubybuntu-6-gedit-3-wtf-set-it-up-for-ruby-web-development

  http://sdbrain.posterous.com/how-to-run-ruby-scripts-from-gedit


::>>Programming Environment<<::

::Git::
You need to install this, no matter what language you're developing in.  

```
$  apt-get install git-core
```


::Ruby::
To install ruby, we'll use rvm because that's the standard rubiest way.  

```
    $  apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf
    $  bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
```

After that, you should be able to close your terminal, and then reopen it, with access to rvm
```
    $  rvm -v
    rvm 1.15.8 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
```

Now you need to install some versions of ruby.  Atm, ruby 1.9.3 exists, so I'll just go ahead and install that.  
```
    $  rvm install 1.9.3      # this will take a while...
```

Before rvm will work in gnome-terminal, you'll need to apply this fix... 

https://rvm.io/integration/gnome-terminal/

After you've done that, you should be able to use ruby...
```
    #  make this version of ruby active by default
    $  rvm --default 1.9.3
    
    #  Manually change to version 1.9.3
    $  rvm use 1.9.3
    
    #  verify which version of ruby you're using
    $  ruby -v
    ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-linux]
```

You may also need to put the below line into your bashrc and possible source it in your .bash_login file.  I did this when I seemed to have encountered a bug.  
(~/.bashrc)
```
    [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
```

Fun!  Want to do a hello world?

(~/dev/ruby/hello/main.rb)
```
    puts "Hello world of Ruby!"
```

Then ruby it, you should see:
```
    $  ruby ~/dev/ruby/hello/main.rb
    Hello world of Ruby!
```



::Mono::
Bla

::Java::
Bla



::>>Invert Screen Colors<<::
Have you ever come to a completely white page on the web and it totally hurts your eyes due to its brightness?  Or maybe you downloaded an application that has a similarly bothersome interface.  Well you can solve these problems easily.

```
    $  apt-get install xcalib
    $  xcalib -invert -alter   # command to invert screen colors
```

https://brutusfacticus.wordpress.com/2012/11/07/how-to-invert-screen-colors-in-ubuntu-based-gnulinux-distros/


###Social


::Record Your Desktop for Screencasting::

```
    $  apt install gtk-recordmydesktop
```



::Voice over IP::
You need to install Jitsi if you'd like to have conversations with people where you don't want intercepts to take place (electronic eves droppers).  

Browse to https://download.jitsi.org/jitsi/debian/ and download the latest stable release. Once it's downloaded, double click the .deb package to install it.  

Now...

::Ekiga

Ekiga is open source and user friendly...  Unlike jitsi witch is way to ugly to constitute friendly.  

```
    apt-get install ekiga
    
    #  This should install zfone for call encryption
    sudo apt-get install libzrtpcpp2
```



::ScreenCasting

RecordMyDesktop is useful, somewhat...  But there's a bug where it makes cinnimon's task bar disapear.  To make it reapear use the show desktop hotkey (ctrl+alt + d).  It also only records .ogv files... which are tough to edit on windows desktops (my Quadcore desktop only has Win7 available to it right now for instance).  

kazam and istanbul and a couple of others do exist too.  











###Backups
http://ubuntuforums.org/showthread.php?t=35087

Steps:
1)  Clear out your caches to make the backup file as small as is needed
2)  Backup all the files
3)  Transfer the backup somewhere safe


To backup your system, you'll want to put everything in your root folder into a tarbal.  But some of the things in your root folder are actually virtual folders, so we need to exclude those.  Below is the conventional command to perform a backup and then a restore.  


1)  Clear out caches

```
    # Clear out apt-get cache of outdated packages
    $  sudo apt-get autoclean
    
    #  The .cache file http://askubuntu.com/questions/102046/is-it-okay-to-delete-cache
    $  ~/.cache
```


Backup:
```
    tar cvpjf backup.tar.bz2 --exclude=/storage --exclude=/proc --exclude=/tmp --exclude=/dev --exclude=/lost+found --exclude=/backup.tar.bz2 --exclude=/mnt --exclude=/sys --exclude=/home/kentos/archive --exclude=/home/kentos/Videos/code /
```

Restore:
```
    tar xvpfj backup.tar.bz2 -C /
```




###Anonymity

::MAC Changer
One of the many things that can identify your computer is your MAC address.  Usually you'll be sitting behind a NAT firewall, so your specific MAC address won't be sent across the internet (instead the MAC address of the router is sent).  But it never hurts to hide your MAC address from the router itself in case the NSA download it's MAC address list.  

First we need to install the app for changing your MAC address.  
```
    $  apt install macchanger
```

The we need to add a line to the pre-up script for changing your MAC right before it switches on at bootup.  I believe this technique will prevent your original MAC address from being exposed to the air.  

(/etc/network/if-pre-up.d/wireless-tools)
```
    .
    .
    .
    sudo macchanger -m 00:00:00:00:00:01 wlan0  # Replace wlan0 with the name 
                                                # of the NIC in `ifconfig` you'd like changed.
                                                # for hard wired ports... You might need put the code
                                                # You can use -r to make it a random value each bootup
```

When your distro upgrades, it may mention that you modified this file.  Overwrite the file with whatever it wants and then mod your MAC address again in the new file.  

To test your work, type ifconfig and see that you h/w address is now 00:00:00:00:00:01 or what ever you set it to.  

::Tor network

To make it difficult for people to see where messages are truely coming from, you can use an Onion Routing network implementation.  The Tor network is completely free, so I advise using that one.  Although it's pretty slow and painful, this is the best way to ensure that no one can find out where you're transmitting messages from.  One caveat is that you should stagger message posting on social outlets to reduce the possibility of applied forensics outing your broadcasting position (which may or may not be a iCafe, or coffee shop far from your home depending on your area of expertise).  

```

```




:: OpenVPN


To be somewhat anonymous you can sign up for some VPNs.  They're pretty cheap, though they're a monthly cost, which can be difficult on people.  Mullvad offers a free 3 hour test for prospective customers if you just wanna check things out with a real VPN provider.  Make sure you get one that supports OpenVPN.  The weaknesses of VPNs are the fact that governments can quietly coerce the owners of VPNs to eliminate the privacy that their customers are paying for.  This can lead the way to law suites, unlawful extortion, or social embarasment (if people find out you're straight or gay in societies where these classifications are socially unpopular).  Most VPNs claim to not follow the rules of tyrannical governments, but in most cases it is not against the law to lie to your customers, and in many cases it is explicitly unlawful to tell one's customers the truth about their level of cooperation with corrupt government policies.  

First install openvpn on your system

```
    sudo apt-get install openvpn
```

The confuguration files for openvpn are placed in `/etc/openvpn`.  If your VPN provider is awesome they will probably supply downloads for configuration files that you can just drop into your openvpn configuration folder.  Once your configuration files are installed, you should be able to control openvpn via `sudo /etc/init.d/openvpn (start, stop, status, etc...)`.  

Once you've got OpenVPN running, you can roughly test whether you're using the VPN by pinging google and seeing that your response time is higher than it ordinarily is.  Your VPN connection is actually very fragile, you'll need to setup some firewall rules in order to prevent traffic from leaking outside of your VPN connection.  

~/scripts/firewall_allow_only_openvpn.sh
```
#!/bin/bash
###########################################
#          Created by Thomas Butz         #
#   E-Mail: btom1990(at)googlemail.com    #
#  Feel free to copy & share this script  #
###########################################

# Adapt this value to your config!
VPN_DST_PORT=1194 #3478

# Don't change anything beyond this point
###########################################

# Check for root priviliges
if [[ $EUID -ne 0 ]]; then
   printf "Please run as root:\nsudo %s\n" "${0}"
   exit 1
fi


# Reset the ufw config
ufw --force reset

# let all incoming traffic pass
ufw default allow incoming
# and block outgoing by default
ufw default deny outgoing

# Every communiction via VPN is considered to be safe
ufw allow out on tun0

# Don't block the creation of the VPN tunnel
ufw allow out $VPN_DST_PORT
# Don't block DNS queries
ufw allow out 53

# Allow local IPv4 connections 
ufw allow out to 10.0.0.0/8
ufw allow out to 172.16.0.0/12
ufw allow out to 192.168.0.0/16
# Allow IPv4 local multicasts
ufw allow out to 224.0.0.0/24
ufw allow out to 239.0.0.0/8

# Allow local IPv6 connections
ufw allow out to fe80::/64
# Allow IPv6 link-local multicasts
ufw allow out to ff01::/16
# Allow IPv6 site-local multicasts
ufw allow out to ff02::/16
ufw allow out to ff05::/16

# Enable the firewall
ufw enable

```

Write that file and then mark it exicutable `sudo chmod 0755 ~/scripts/firewall_allow_only_openvpn.sh' then you'll be able to execute it `./firewall_allow_only_openvpn.sh`


Note to control your firewall, use these commands:
```
    $  sudo ufw disable
    $  sudo ufw enable
```







###Hacking

Cinnamon's system settings dialog menu can be hacked here:
/usr/lib/cinnamon-settings









::>> Fix File Permissions <<::

```
    find . -type d -print0 | xargs -0 chmod 0775 # For directories
    find . -type f -print0 | xargs -0 chmod 0664 # For files
```





