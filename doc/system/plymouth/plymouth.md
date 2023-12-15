plymouth
==============


## get plymouth debug log

1.  vim /etc/default/grub and add `plymouth:debug` in GRUB_CMDLINE_LINUX_DEFAULT 
2.  debug log  file `/var/log/plymouth-debug.log`

## debug plymouth in post boot


1. Boot system and login as usual
2. Install plymouth-x11 package (allows you to see the boot screen in an X11 window)
    - sudo apt-get install plymouth-x11

3. Start a terminal 
4. Start the Plymouth daemon by running the following:
    - sudo plymouthd --debug --tty=`tty` --no-daemon
5.  plymouth client running : 
	 plymouth --show-splash 

quit debug 

>  sudo plymouth --quit





## reference connection

##  ubuntu wiki

	https://wiki.ubuntu.com/Plymouth


