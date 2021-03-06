# System Inventory

### Reviewed by:

## Overview



This feature refers to our ability to correctly identify the operating system, version, and virtualization capabilities/status of hosts registered to Spacewalk. 
## Requirements



 1. Identify the following operating systems and versions:
  1. Red Hat Enterprise Linux: 4 (AS, ES, ...), 5 (Server, Desktop)
  1. Fedora (10+)
  1. CentOS (4, 5)
  1. Solaris (*TODO*: what versions do we want to support?)
  1. "Unknown" system (for everything we can't identify)
 1. System architecture: i386, i686, x86_64, ppc, s390x, etc. 
 1. Virtualization Status
  1. Virtualized guest (Xen, KVM, VMware)
    1. Can we also detect IFL, and Hyper-V guests?
    1. Identify and associate with host system.
      1. '''TODO''': Is this possible? Safe?
      1. Deal with scenario where host system is not registered.
      1. Deal with scenario where host system is registered but not to the same Spacewalk/Satellite.
  1. Virtualization capable host (Xen full/para-virt, KVM full-virt)
 1. Idenfity number of CPUs and CPU cores.
  * _Pretty sure we already have this, see current hardware info example below. cat /proc/cpuinfo on a multi-cpu multi-core system looks like we'll have enough info to work with._ - dgoodwin
 1. Allow searching of this data in web UI.
  1. *TODO*: Need examples of specific searches to see where to go with this. Take a look at current advanced search screen to see what we have to work with.
 1. API should allow mining of data for permutiations of hardware, OS, virt capabilities/state, and system grouping (?).
  1. *TODO*: Need examples.
### Question: Two Birds with One Stone?



Have we looked at how the push for a stronger Configuration Management Framework may support this? Cross requirement integration is good right? :)

* Given that Puppet is a strong candidate for doing Configuration Management - perhaps a lot of this work falls into our lap for 'free'?
 * [[http://www.puppetlabs.com/puppet/related-projects/facter/]] : Gives all the data, 'Facts' and more
 * [[http://www.puppetlabs.com/puppet/related-projects/dashboard/]] : Makes it pretty and printable

The 'Reporting Framework' issue may also be addressed via extension?

Side Thoughts:

* Pros
  * Puppet worries about the supported architectures (RHEL, OSX, Solaris, Windows, AIX, Ubuntu, etc.) - so that we can just 'consume'
  * Puppet has a REST API and Python/Perl interfaces
  * The Data / System Reports are in yummy YAML
  * I think the 'Facts' concept could really add some punch for customers/end-users to 'self service' on defining and redefining what they want Satellite/Spacewalk to be telling them

* Cons
  * Ruby not Python ( but has Python APIs - [kernel is C right? ;) ](The)
  * Possible 'stretch'/'function creep' of what Puppet / Facter are designed for ? ( Good? / Bad? )
  * Not in [[http://www.fedorahosted.org]] :)

--aaronlippold (lippold@gmail.com)
## Other Feature Impact



TBD
## Proposed Implementation

### Packaging


 1. Add dependency on smolt to rhn-setup.

  * Must deal with issue where smolt currently sets up cron job in post script to re-submit hardware details to an actual smolt server (called smoon) monthly. We do not want this, we're just aiming to leverage the client side bits. Maybe able to split smolt from say libsmolt upstream.
  * Will need this slightly modified smolt rpm available in stock RHEL.
   * _Will this be a problem?_ - dgoodwin
### Client Side Tools



 1. Modify rhn-setup code replacing existing hardware detection bits with Smolt data.
  * Note that smolt's code was originally based on ours.
  * Most of our hardware detection happens where we instantiate a "hardware.Hardware" object.
 1. Submit data to Spacewalk. 
  * I believe we must make sure that even after leveraging smolt code to lookup hardware data, what we submit must appear in *exactly* the same format as it used to for the benefit of RHN hosted. We can likely add additional information, but probably not take anything away. 
  * Data submission happens in rhnreg.sendHardware and is received on the server in registration.add_hw_profile.
### Database



 1. Introduce new schema to store any new information.
  * Most of the hardware information is stored already, it's primarily the new OS/Version info we'll need to add on.
### Backend



 1. Accept the new incoming data, populate database. Hooking into registration process should cover us for fresh registrations and kickstart re-registrations. Do we also need to periodically refresh for hardware changes? (this probably happens already but need to verify)
 1. Information should be available before doing entitlement checks, but rhnreg_ks sends it after registering...
### Web UI

 1. Under Systems -> (system) -> Details -> Overview in the "System Info" section:

  1. Add "Operating System". (Fedora 10, Red Hat Enterprise Linux 5 Server, CentOS 5)
  1. Already display if system if a para-virt host/guest, update to include include KVM and clearly differentiate between the two.
 1. Under Systems -> (system) -> Details -> Hardware:
  1. Update General section with number of CPU cores.
  1. Add virtualization section, display virtualization capabilities.  (KVM, Xen full-virt, Xen para-virt)
 1. Allow for searching/filtering, expand search interface "Field to Search" to include:
  1. Operating System
  1. Virtualization Capabilities (Xen/KVM?)
  1. Guest systems (somehow...)
  1. Host systems (somehow)
 1. Allow admins to specify custom channels as the default for a particular version of an operating system.
  * This would allow Spacewalk users to create their Fedora or Cent OS channels, and then automatically register systems to then without the need for an explicit activation key. (similar to the way we auto-subscribe to RHEL channels in Satellite, which may also be a candidate for cleanup)
### API

 1. Allow for searching for various permutations of the new pieces of inventory information. 

 1. *TODO*: Specific API calls to implement.
### Known Issues



 * How do we populate this information for pre-existing systems on an upgraded install?
 * Can an administrator manually override the data our code determines to be accurate in the UI?
### Future Enhancements
## Mock Ups



*TODO*
## Tasks





|  Description  |  Estimate  |  Confidence  |
| --- | --- | --- | --- |
|    |   |   |  |
|    |   |   |  |
## Tests

 * RHN client tools do not currently have much in the way of unit test framework, these will largely be left to manual testing.

 * Backend code unit tests can be used to cover new logic. 
 * Java web UI and API can be tested using existing unit test and API test framework respectively.
## Risks



 * If we go the route of re-using smolt, will probably need to submit some patches to add some of the data we're interested in.
 * Replacing our hardware/OS detection with the more recent smolt code would require a bit of work and a lot of testing in the client side tools, CLI, GUI, and TUI.
  * What happens with respect to hosted if we start messing around with the client side packages? Information going up to the server may have to remain exactly the same, although perhaps with additions which the server can gracefully ignore if it's not ready for the new.
 * Is smolt the best option, or should we just hack onto what we have, perhaps just using pieces of their code? Worth noting that guest detection will probably need to be added upstream in smolt unless we're content inferring it based on other info in the profile already as we do today.
 * There are *no EL4 smolt packages* in EPEL as there are for RHEL 5. Why?
## Additional Info



Some smolt data samples:

Fedora 10 KVM guest:


    [root@sw2 ~]# smoltSendProfile 
    	UUID: 05668b04-abdb-4306-8196-c101b1223319
    	OS: Fedora release 10 (Cambridge)
    	Default run level: 3
    	Language: en_US.utf8
    	Platform: i686
    	BogoMIPS: 4788.27
    	CPU Vendor: GenuineIntel
    	CPU Model: QEMU Virtual CPU version 0.10.5
    	Number of CPUs: 1
    	CPU Speed: 2393
    	System Memory: 1008
    	System Swap: 2047
    	Vendor: Unknown
    	System: Unknown
    	Form factor: unknown
    	Kernel: 2.6.27.25-170.2.72.fc10.i686
    	SELinux Enabled: False
    	SELinux Policy: targeted
    	SELinux Enforce: Unknown
    
    		 Devices
    		=================================
    		(None:None:None:None) platform, i8042, None, Platform Device (i8042)
    		(None:None:None:None) platform, pcspkr, None, Platform Device (pcspkr)
    		(None:None:None:None) input, Unknown, None, Power Button (FF)
    		(4115:184:6900:4352) pci, Unknown, VIDEO, GD 5446
    		(None:None:None:None) pnp, rtc_cmos, None, AT Real-Time Clock
    		(None:None:None:None) input, Unknown, MOUSE, ImExPS/2 Generic Explorer Mouse
    		(6900:4096:6900:1) pci, virtio-pci, NETWORK, Virtio network device
    		(None:None:None:None) input, Unknown, MOUSE, Macintosh mouse button emulation
    		(None:None:None:None) scsi_generic, Unknown, None, SCSI Generic Interface
    		(6900:4097:6900:2) pci, virtio-pci, OTHER, Virtio block device
    		(6900:4098:6900:5) pci, virtio-pci, OTHER, Unknown (0x1002)
    		(None:None:None:None) input, Unknown, None, PC Speaker
    		(None:None:None:None) tty, Unknown, None, 16550A-compatible COM port
    		(None:None:None:None) serio, psmouse, None, i8042 AUX port
    		(None:None:None:None) platform, serial8250, None, Platform Device (serial8250)
    		(32902:28688:6900:4352) pci, ata_piix, IDE, 82371SB PIIX3 IDE [Natoma/Triton II]
    		(32902:28947:6900:4352) pci, piix4_smbus, OTHER, 82371AB/EB/MB PIIX4 ACPI
    		(None:None:None:None) serio, atkbd, None, i8042 KBD port
    		(None:None:None:None) scsi, sr, None, SCSI Device
    		(None:None:None:None) net, Unknown, NETWORK, Bridge Interface
    		(None:None:None:None) scsi_host, Unknown, None, SCSI Host Adapter
    		(None:None:None:None) input, Unknown, KEYBOARD, AT Translated Set 2 keyboard
    		(None:None:None:None) platform, Unknown, None, Platform Device (vesafb.0)
    		(None:None:None:None) pnp, serial, None, 16550A-compatible COM port
    		(None:None:None:None) pnp, i8042 aux, None, PS/2 Port for PS/2-style Mice
    		(None:None:None:None) net, Unknown, NETWORK, Loopback device Interface
    		(32902:4663:6900:4352) pci, Unknown, OTHER, 440FX - 82441FX PMC [Natoma]
    		(32902:28704:6900:4352) pci, uhci_hcd, USB, 82371SB PIIX3 USB [Natoma/Triton II]
    		(32902:28672:6900:4352) pci, Unknown, OTHER, 82371SB PIIX3 ISA [Natoma/Triton II]
    		(None:None:None:None) scsi_host, Unknown, None, SCSI Host Adapter
    		(None:None:None:None) net, Unknown, NETWORK, Networking Interface
    		(None:None:None:None) scsi_host, Unknown, None, SCSI Host Adapter
    		(None:None:None:None) pnp, i8042 kbd, None, IBM Enhanced (101/102-key, PS/2 mouse support)
    		(None:None:None:None) virtio, virtio_blk, None, VirtIO Device (virtio1)
    		(None:None:None:None) virtio, virtio_net, None, VirtIO Device (virtio0)
    		(None:None:None:None) virtio, virtio_balloon, None, VirtIO Device (virtio2)
    
    	Filesystem Information
    		device mtpt type bsize frsize blocks bfree bavail file ffree favail
    		===================================================================
    		/dev/root / ext3 4096 4096 9740660 7798251 7303455 2473984 2399106 2399106
    		/dev/vda1 /boot ext3 1024 1024 194442 174442 164403 50200 50160 50160


RHEL 5 Physical Host:


    [root@rlx-3-12 ~]# smoltSendProfile 
    	UUID: d48ed2dd-b97a-411f-84d2-dd22305b2ccd
    	OS: RedHatEnterpriseServer 5.3 Tikanga
    	Default run level: 3
    	Language: en_US.UTF-8
    	Platform: i686
    	BogoMIPS: 5980.16
    	CPU Vendor: GenuineIntel
    	CPU Model: Mobile Intel(R) Pentium(R) III CPU - M  1200MHz
    	Number of CPUs: 1
    	CPU Speed: 1493
    	System Memory: 1770
    	System Swap: 996
    	Vendor: RLX Technologies
    	System: SB 1200i 0A07
    	Form factor: unknown
    	Kernel: 2.6.18-128.el5xen
    	SELinux Enabled: True
    	SELinux Policy: targeted
    	SELinux Enforce: Permissive
    
    		 Devices
    		=================================
    		(None:None:None:None) platform, i8042, None, Platform Device (i8042)
    		(None:None:None:None) platform, pcspkr, None, Platform Device (pcspkr)
    		(4281:28929:5961:4160) pci, ali1535_smbus, OTHER, M7101 Power Management Controller [PMU]
    		(None:None:None:None) platform, Unknown, None, Platform Device (bluetooth)
    		(None:None:None:None) input, Unknown, None, PC Speaker
    		(4932:13104:0:0) pci, Unknown, OTHER, Unknown (0x3330)
    		(None:None:None:None) platform, Unknown, None, Platform Device (vesafb.0)
    		(4281:5427:5961:4112) pci, Unknown, OTHER, M1533/M1535/M1543 PCI to ISA Bridge [Aladdin IV/V/V+]
    		(None:None:None:None) serio, Unknown, None, i8042 KBD port
    		(32902:4649:5961:8208) pci, e100, NETWORK, 82557/8/9/0/1 Ethernet Pro 100
    		(None:None:None:None) pnp, i8042 aux, None, PS/2 Port for PS/2-style Mice
    		(32902:4649:5961:8208) pci, e100, NETWORK, 82557/8/9/0/1 Ethernet Pro 100
    		(4932:13089:4162:4162) pci, Unknown, OTHER, Unknown (0x3321)
    		(4932:13088:4162:4162) pci, Unknown, OTHER, Unknown (0x3320)
    		(None:None:None:None) ide, ide-disk, None, IDE device (master)
    		(None:None:None:None) net, Unknown, NETWORK, Networking Interface
    		(None:None:None:None) net, Unknown, NETWORK, Networking Interface
    		(None:None:None:None) net, Unknown, NETWORK, Networking Interface
    		(4281:21033:5961:4128) pci, ALI15x3_IDE, IDE, M5229 IDE
    		(32902:4649:5961:8208) pci, e100, NETWORK, 82557/8/9/0/1 Ethernet Pro 100
    		(None:None:None:None) pnp, i8042 kbd, None, IBM Enhanced (101/102-key, PS/2 mouse support)
    
    	Filesystem Information
    		device mtpt type bsize frsize blocks bfree bavail file ffree favail
    		===================================================================
    		/dev/root / ext3 4096 4096 13886985 13618579 12901779 14338368 14303111 14303111
    		/dev/hda1 /boot ext3 1024 1024 194442 181309 171270 50200 50164 50164
    

And an example of the hardware data we receive today, this is from a Centos 5 KVM guest:


    [{'bus': 'MISC',
      'class': 'KEYBOARD',
      'desc': 'AT Translated Set 2 keyboard',
      'detached': 0,
      'device': 'input/event0',
      'driver': 'unknown',
      'pciType': -1},
     {'bus': 'MISC',
      'class': 'MOUSE',
      'desc': 'ImExPS/2 Generic Explorer Mouse',
      'detached': 0,
      'device': 'input/event1',
      'driver': 'unknown',
      'pciType': -1},
     {'bus': 'pci',
      'class': 'OTHER',
      'desc': 'Intel Corporation|440FX - 82441FX PMC [Natoma]',
      'detached': 0,
      'driver': 'unknown',
      'pciType': 1},
     {'bus': 'pci',
      'class': 'OTHER',
      'desc': 'Intel Corporation|82371SB PIIX3 ISA [Natoma/Triton II]',
      'detached': 0,
      'driver': 'unknown',
      'pciType': 1},
     {'bus': 'pci',
      'class': 'IDE',
      'desc': 'Intel Corporation|82371SB PIIX3 IDE [Natoma/Triton II]',
      'detached': 0,
      'driver': 'PIIX_IDE',
      'pciType': 1},
     {'bus': 'ide',
      'class': 'HD',
      'desc': 'QEMU HARDDISK',
      'detached': 0,
      'device': 'hda',
      'driver': 'unknown',
      'pciType': -1},
     {'bus': 'ide',
      'class': 'CDROM',
      'desc': 'QEMU DVD-ROM',
      'detached': 0,
      'device': 'hdc',
      'driver': 'unknown',
      'pciType': -1},
     {'bus': 'pci',
      'class': 'USB',
      'desc': 'Intel Corporation|82371SB PIIX3 USB [Natoma/Triton II]',
      'detached': 0,
      'driver': 'uhci_hcd',
      'pciType': 1},
     {'bus': 'usb',
      'class': 'OTHER',
      'desc': 'USB Hub Interface',
      'detached': 0,
      'driver': 'hub',
      'pciType': -1},
     {'bus': 'pci',
      'class': 'OTHER',
      'desc': 'Intel Corporation|82371AB/EB/MB PIIX4 ACPI',
      'detached': 0,
      'driver': 'piix4_smbus',
      'pciType': 1},
     {'bus': 'pci',
      'class': 'VIDEO',
      'desc': 'Cirrus Logic|GD 5446',
      'detached': 0,
      'driver': 'unknown',
      'pciType': 1},
     {'bus': 'pci',
      'class': 'OTHER',
      'desc': 'Realtek Semiconductor Co., Ltd.|RTL-8139/8139C/8139C+',
      'detached': 0,
      'driver': '8139cp',
      'pciType': 1},
     {'bus': 'MISC',
      'class': 'NETWORK',
      'desc': 'Networking Interface',
      'detached': 0,
      'device': 'eth0',
      'driver': 'unknown',
      'pciType': -1},
     {'bus': 'pci',
      'class': 'AUDIO',
      'desc': 'Ensoniq|ES1370 [AudioPCI]',
      'detached': 0,
      'driver': 'ENS1370',
      'pciType': 1},
     {'bus': 'pci',
      'class': 'OTHER',
      'desc': 'Qumranet, Inc.|Unknown (0x1002)',
      'detached': 0,
      'driver': 'unknown',
      'pciType': 1},
     {'bogomips': '4803.35',
      'cache': '2048 KB',
      'class': 'CPU',
      'count': 1,
      'desc': 'Processor',
      'model': 'QEMU Virtual CPU version 0.10.5',
      'model_number': '6',
      'model_rev': '3',
      'model_ver': '2',
      'other': 'fpu de pse tsc msr pae mce cx8 apic mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall lm up pni',
      'platform': 'i386',
      'speed': 2393,
      'type': 'GenuineIntel'},
     {'class': 'MEMORY', 'ram': '249', 'swap': '511'},
     {'class': 'NETINFO', 'hostname': 'unknown', 'ipaddr': '192.168.1.131'},
     {'asset': '(system: Not Specified) ',
      'bios_vendor': 'QEMU',
      'bios_version': 'QEMU',
      'class': 'DMI',
      'system': 'Not Specified Not Specified',
      'vendor': 'Not Specified'},
     {'class': 'NETINTERFACES',
      'eth0': {'broadcast': '192.168.1.255',
               'hwaddr': '54:52:00:1e:5b:a8',
               'ipaddr': '192.168.1.131',
               'module': '8139cp',
               'netmask': '255.255.255.0'},
      'lo': {'broadcast': '0.0.0.0',
             'hwaddr': '00:00:00:00:00:00',
             'ipaddr': '127.0.0.1',
             'module': 'loopback',
             'netmask': '255.0.0.0'},
      'sit0': {'broadcast': '',
               'hwaddr': '00:00:00:00:00:00',
               'ipaddr': '',
               'module': 'Unknown',
               'netmask': ''}}]

