# vSPC.py

A server implementing the VMware VSPC serial port multiplexer.  We use it
to get text mode "serial" consoles from our Linux VMs.

This is a fork of vSPC.py by Zach Loafman / Isilon Systems.


## Origins

It was forked from `git://git.code.sf.net/p/vspcpy/code` in late 2014,
at commit 80c3a4b228562351b0eead91a1d99d41aa850fc1 from Sep 6 2011.

You can find the original README at 
[vspcpy on SourceForge](http://sourceforge.net/p/vspcpy/home/Home/).

Given that sourceforge is a rather dubious site now at best I suggest some
caution when visiting it.

Note that there are several significantly reworked versions of vSPY.py where the files are split off into a lib directory. Some bugfixes have been incorporated from those ports.


## Changes from vSPC.py

1. Add SSL support and --ssl, ssl-cert and --ssl-key options.
1. Changes default ports to proxy=6779 and admin=6780.
1. Syslog output shows the daemon name and PID.
1. Logs a lot more to INFO priority.
1. Add --verbose separate from --debug to log verbosely to syslog but
   still daemonize.
1. Fix a serious bug where VMs wouldn't migrate if the generated secret
   contained an IAC (\0xff) character.


## Using It

Start the daemon somewhere in server mode.  There is no security or
authentication on the client connections, so I strongly suggest you
start the client ("admin") listeners exclusively on 127.0.0.1.

E.g.,
```
vSPC.py --server --proxy :6779 --admin 127.0.0.1:6780
```

Or with SSL:
```
vSPC.py --server --proxy :6779 --admin 127.0.0.1:6780 \
        --ssl \
        --ssl-cert /etc/ssl/certs/my.cert \
        --ssl-key /etc/ssl/private/my.key
```

On your VMs, add a serial port device configured as "Use Network" with
settings like this:

* Status: Connect At Power On
* Direction: Server
* Port URI: vSPC.py
* Use Virtual Serial Port Concentrator
* vSPC URI: telnet://myhost.example.org:6779
* use 'telnets://' for SSL

When a VM powers on VMware will initiate a connection to vSPC.py that's plumbed to the virtual serial port on the VM.  vSPC.py will proxy between that and a listening TCP port on localhost.

Then you can run vSPC.py in admin mode to list the VMs connected:

```
vSPC.py localhost:6780
```

You will get output like this, with three colon-separated fields:
```
vm-name-1:500d1f755b7d4870-e7dd86d3fcda096a:50007
vm-name-2:500d9a712d07349b-f6f3bc7110617dea:50300
vm-name-3:500d7ac1f0e0028a-c47f7b3473c3ca2e:50331
```

Where vm-name-x is the name of a VMware virtual machine, the midle field
is VMware's UUID for the VM, and the final field is the listening TCP port
number.

To get the console for a given VM telnet to its port on localhost.
