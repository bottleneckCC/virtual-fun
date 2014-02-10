Xen on Debian dom0
====

Installed Xen with Debian as the dom0 by following :

<http://wiki.xenproject.org/wiki/Xen_Beginners_Guide>


Squid caching server
-----


Got Xen installed, but still can't create an Ubuntu domU.

The first time I tried to install 12.04LTS it couldn't find the network hardware.  And since I'm running a net boot I don't think I can change the config file.

It had to be something with my Xen networking config because the Ubuntu installer should see the xenbr0 as the interface, righ?

There was a typo in my /etc/network/interface.  My file was:
	
	auto lo
	iface lo inet loopback

	auto xenbr0
	iface xenbr0 inet dhcp
    	bridge-ports eth0

	auto eth0
	iface eth0 inet manual
	
It should be:

	auto lo
	iface lo inet loopback

	auto xenbr0
	iface xenbr0 inet dhcp
    	bridge_ports eth0

	auto eth0
	iface eth0 inet manual
	
I can't remember what I did last time so I'm going to have to start over using the instructions found here : <https://help.ubuntu.com/community/Xen>


Everything went well up to the disk partitioning this time.  Even that seemed to work, but I realized that I created the volume to only be 5 gigs.  Oh well, I guess it's only a test.

My /etc/xen/ubuntu.cfg :
	
	name = "ubuntu"

	memory = 8192

	disk = ['phy:/dev/vg0/ubuntu,xvda,w']
	vif = ['bridge=xenbr0']

	kernel = "/var/lib/xen/images/ubuntu-netboot/vmlinuz"
	ramdisk = "/var/lib/xen/images/ubuntu-netboot/	initrd.gz"
	extra = "debian-installer/exit/always_halt=true -- \		console=hvc0"


Nice. Just booted up the Ubuntu server and logged in.  Now to install Squid.

	$ sudo apt-get install squid3
	

squid3 insalled.  Now to configure.

Following :
<http://cottagedata.com/security/squid/squid.html?book_num=4&chapter_num=1>

Using the instructions from that site I am getting an Access Denied error from the squid proxy server.

I just had to reload squid:

	$ sudo /etc/init.d/squid3 reload
	
Useful to see if cache is working:

	$ sudo tail -f /var/log/squid3/access.log
	

