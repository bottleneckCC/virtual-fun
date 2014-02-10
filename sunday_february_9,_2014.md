Log
====

### Sunday February 9, 2014

I think I broke the plex server.  I had everything set up, but the test machine I created had only 512MB of RAM and only 10G of storage space.  So when I attempted to play a video from the plex media server it started then halted and hung up.

I tried to create a new virtual machine, but thngs wet awry.  I may have to start over, but I am successful in that I installed SmartOS on a computer, created and started an Ubuntu virtual machine, installed Plex Media Server on that virtual machine and it was "functional."

Yeah there's no space left on the device and so I can't even log back into SmartOS after a reboot.

Hardware
----
hardware specs here

.json
---
	{
  		"alias": "ubuntu12",
  		"hostname": "ubuntu12",
  		"brand": "kvm",
  		"vcpus": 1,
  		"autoboot": false,
  		"ram": 4096,
  		"resolvers": [ "10.1.1.1" ],
  		"disks": [
    		{
      		"boot": true,
      		"model": "virtio",
      		"size": 40960
    		}
  		],
  		"nics": [
    		{
      		"nic_tag": "admin",
      		"model": "virtio",
      		"ip": "10.1.1.11",
      		"netmask": "255.255.255.0",
      		"gateway": "10.1.1.1",
      		"primary": true
    		}
  		]
	}
	
Could it be a degraded hard drive?  I think I am using a teacher's old hard drive that was fucked to begin with.

I do think there is something going on with the size of RAM that I want to use, but there is a problem I keep running into where I can't even create zones.  Maybe this  has to do with a corrupted drive.

Started over again, this time using only disk 0 and not 0 and 6.

I created another zone using the previous json ("ubuntuPlex.json") changing the 'size' parameter to 10240.

Still starting and shutting down immediately.

Changed the RAM to 512 and it's good.

Next I'm going to try 1024MB RAM.

Cool, 1024MB worked.  So now to try 2048.

2048 is too much.  It shuts down.  Well, let's see how slow an ubuntu plex media running 1024 is.

Seems a lot better already, but 1024 isn't very fast.

Plex Media Server
-----
First I had to change the DNS of my ubuntu sever because it couldn't find the sources.

Then I had to apt-get update.

That let me install ahavi-daemon, but I still couldn't *apt-get install* plex media server so I had to download the *.deb* and then there were dependencies.  So I had to *apt-get install ahavi-utils*.  It gave me an error again , but after *apt-get -f install* everything fixed itself.

It's way too slow. I don't know what I was thinking.  I guess I was just eager to get it off the ground, but if I have to run it this slow it won't work.  If it can be a percentage of the total memory (e.g. 25% of 16GB) then I would be ok.  But 1024MB is obviously not going to cut it.

I take that back it's running at 2048.  Well, I think it is anyway.  I could have sworn I had to change it back, but the **ubuntuPlex.json** file has 2048 set for RAM.  This is the file I believe the new zone was created off of.  Also, I noticed that the cap for the new zone was 2048MB after I ran:

	[root@50-46-5d-b3-5d-d3 /var]# zonememstat
	
and got the following output:

    ZONE  		RSS(MB)  CAP(MB)    NOVER  POUT(MB)
    global      91        -          -         -
 	0229e545…   1423     2048        0         0	
	
Nope.  I changed the file after I had started the first one running on 1024MB.  Damnit.

Trying:

	{
  	"brand": "kvm",
 	 "resolvers": [
   	"10.0.0.1",
   	"8.8.8.8"
  	],
  	"ram": "8192",
  	"vcpus": "1",
  	"nics": [
   	{
        "nic_tag": "admin",
        "ip": "10.0.0.22",
        "netmask": "255.255.255.0",
        "gateway": "10.0.0.1",
        "model": "virtio",
        "primary": true
   	}
 	],
       "disks": [
        {
         "image_uuid": "bad2face-8738-11e2-	ac72-0378d02f84de",,
          "boot": true,
          "model": "virtio",
          "image_size": 10240
        }
 		]
	}


from : <http://wiki.smartos.org/display/DOC/How+to+create+a+KVM+VM+(+Hypervisor+virtualized+machine+)+in+SmartOS>

and trying ubuntu-12.04 image

~       

Works:

	{
	"brand": "joyent",
	"image_uuid": "bad2face-8738-11e2-ac72-0378d02f84de",
  	"alias": "ubuntuTest",
  	"hostname": "ubuntuTest",
 	 "max_physical_memory": 8192,
 	 "quota": 20,
	  "nics": [
 	  {
       	 "nic_tag": "admin",
       	 "model": "virtio",
       	 "ip": "10.0.0.21",
      	  "netmask": "255.255.255.0",
      	  "gateway": "10.0.0.1"
   	}
 		]
	}
~    
but using the latest smartos base64 image    

from: <http://wiki.smartos.org/display/DOC/How+to+create+a+zone+%28+OS+virtualized+machine+%29+in+SmartOSz>


OK it's at least being created (the latter code).  I need to change "disk_size" to "size"

Hmm.  Time out.
	
	[root@50-46-5d-b3-5d-d3 ~]# vmadm create -f 	ubuntuTest.json
	timed out waiting for /var/svc/provisioning to move 	for 8cf7f11e-96d3-401c-8022-8414674aa6c1

Trying this now:

	{
 	 "alias": "testubuntu12",
 	 "hostname": "testubuntu12",
 	 "brand": "kvm",
		  "vcpus": 1,
	  "ram": 8192
		,
  	"resolvers": [ "10.0.0.1" ],
 	 "disks": [
   	 {
      	"image_uuid": 	"1327ed36-5130-11e2-95a8-9b5a153adf3e",
     	 "boot": true,
     	 "model": "virtio",
    	  "size": 10240
	    }
	  ],
 	 "nics": [
   	 {
     	 "nic_tag": "admin",
     	 "model": "virtio",
     	 "ip": "10.0.0.22",
    	  "netmask": "255.255.255.0",
    	  "gateway": "10.0.0.1",
    	  "primary": true
   	 }
  	]
	}

It's basically the one with the uuid of the image inserted and autoboot:off line removed.


Here's ubuntuPlex.json that I could get working, but only under 1024M RAM:

		{
 	 "alias": "ubuntu12",
 	 "hostname": "ubuntu12",
 	 "brand": "kvm",
  	"vcpus": 1,
  	"autoboot": false,
  	"ram": 8192
	,
  	"resolvers": [ "10.0.0.1" ],
  	"disks": [
   	 {
      	"boot": true,
      	"model": "virtio",
     	 "size": 10240
    	}
  	],
  	"nics": [
   	 {
      	"nic_tag": "admin",
     	 "model": "virtio",
      	"ip": "10.0.0.16",
     	 "netmask": "255.255.255.0",
     	 "gateway": "10.0.0.1",
     	 "primary": true
    	}
 	 ]
	}

Kept getting:

	# vmadm create -f test2.json
	timed out waiting for /var/svc/provisioning to move 	for dbcb2dcf-33a0-4352-87c5-0b8408cb5d00
	
so I changed:

	"primary": true	
	
to:

	"primary": "true"	

Now I'm getting:

	# vmadm create -f test2.json
	Command failed: cannot create 'zones/	83237b34-6dc6-4ea5-a8a9-213d4450b412-disk0': out of space

	
Now I'm going to try a clean install with the test2.json which I will post in the repository.

	
