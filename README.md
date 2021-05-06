# BSD-Asterisk
## Vagrant + VirtualBox

1. Select order in Cmder oder cmd:
   ```
   cd D:\Projects\VirtualOS\FreeBSD
   ```

2. Initialisieren FreeBSD durch Vagrant:
   ```
   vagrant init freebsd/FreeBSD-11.0-STABLE
   ```

3. Installieren FreeBSD
   ```
   vagrant up
   ```

4. SSH Zugriff
   ```
   vagrant ssh
   ```
   
## Create asterisk VoIP-Server in FreeBSD

1. Install Asterisk

   ```
   pkg install asterisk13
   ```

   to run server automatically on startup add the following line into ```/etc/rc.conf```:<br>
   ```
   asterisk_enable="YES"
   ```

   start the server:

   ```
   /usr/local/etc/rc.d/asterisk start
   ```

   check if server is running:

   ```
   ps ax | grep asterisk
   ```

3. Initial Configuration

   location of the config-files:

   ```
   /usr/local/etc/asterisk/
   ```

   basic sip.conf example:

   ```
   [general]
   context=unauthenticated ; default context for incoming calls
   allowguest=no ; disable unauthenticated calls
   srvlookup=yes ; enabled DNS SRV record lookup on outbound calls
   udpbindaddr=0.0.0.0 ; listen for UDP requests on all interfaces
   tcpenable=no ; disable TCP support
   
   [office-phone](!) ; create a template for our devices
   type=friend ; the channel driver will match on username first, IP second
   context=LocalSets ; this is where calls from the device will enter the dialplan
   host=dynamic ; the device will register with asterisk
   nat=yes ; assume device is behind NAT
   secret=123; a secure password for this device -- DON'T USE THIS PASSWORD!
   dtmfmode=auto ; accept touch-tones from the devices, negotiated automatically
   disallow=all ; reset which voice codecs this device will accept or offer
   allow=ulaw ; which audio codecs to accept from, and request to, the device
   allow=alaw ; in the order we prefer
   allow=all
   textsupport=yes
   [alice](office-phone)
   [bob](office-phone) 
   ```

   basic extension.conf example:

   ```
   [general]
   [LocalSets]
   exten => 99,1,VoiceMailMain()
   exten = 1337,1,Answer()
   same = n,Wait(1)
   same = n,Playback(hello-world)
   same = n,Hangup()
   
   exten = 2000, 1, Dial(SIP/alice)
   exten = 2001, 1, Dial(SIP/bob)
   ```

5. Monitoring and Controlling

   enter asterisk CLI with the following command:

   ```
   asterisk -r
   ```

   show all sip user:

   ```
   sip show peers
   ```

   show all call connections:

   ```
   core show channels
   ```

   Find useful commands:

   ```
   core show help
   ```

   Get info how to use a command:

   ```
   core show help sip show channel
   ```

	Reload configuration files:
	
	```
	sip reload     # reload sip.conf 
	iax2 reload    # reload iax.conf 
	core reload    # reload entire configuration
	```
	
	Stop server:
	
	```
	core stop now
	```
	
	Getting useful informations:
	
	```
	sip show channel     # Show detailed SIP channel info
	sip show domains     # List our local SIP domains
	sip show history     # Show SIP dialog history
	```

6. Debuggin and logs

   logfile location can be configured in asterisk.conf

   edit logger.conf to enalbe specific debug output to your filesystem:

   ```
   [logfiles]
   debug_log_123456 => notice,warning,error,debug,verbose,dtmf
   ```
   Enable/Disable live debug in CLI:
   ```
   sip set debug on/off
   iax2 set debug on/off
   ```

8. iax.conf

   Server A iax.conf example:
   ```
   [general]
   [ast2]
   host=192.168.0.XX
   type=friend
   trunk=yes
   qualify=yes
   secret=1234
   context=LocalSets
   ```
   Server B iax.conf example:
   ```
   [general]
   [ast1]
   host=192.168.0.XX
   type=friend
   trunk=yes
   qualify=yes
   secret=1234
   context=LocalSets
   ```
   Dialplan example:
   ```
   exten => 09021001,1,Dial(IAX2/ast2/1001)
   ;          ^   ^               ^     ^
   ;          |   |               |     |
   ; virt.prefix ext      connection   ext
   ```
   SIP-User with the number 1001 on Server B is callable with the number 09021001

## 🎉 Credits

- [Andrej Kopp](https://github.com/sequelone)
