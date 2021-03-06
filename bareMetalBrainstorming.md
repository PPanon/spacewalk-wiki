
# **DEPRECATED, NO LONGER USED**

# Bare Metal Kickstarting - Best Practices?
 


This page exists to throw some ideas around as to different approaches to the problem of bare metal kickstarting in a large scale production environment.  From a Spacewalk perspective, this is coming up now as we're in the process of adding Cobbler's considerable provisioning mechanics to Spacewalk's core toolset, and as we walk up to it we see that the road is perhaps not as clearly defined as we had thought.

And mostly, it comes down to workflow in the context of "what really happens in the real world".

An example: Imagine you've got a production deployment of two hundred machines divided amongst five tiers, each on separate network segments - and within each of the tiers say there are maybe two or three different machine types.  (And frankly - part of the problem is - is that even a realistic picture of a production environment - we've not done a customer survey to answer the question, so it's just a guess at what we imagine would be a pretty complex deployment.)

Given that architecture, you've got a two main workflows: initial setup, and maintenance.

Initial setup includes a bunch of truly one-time stuff like network wiring and configuration, rack installation, power distribution, etc.  And then comes time to add computers to all this.  These must be provisioned and physically installed - not necessarily in that order.  And once you're infrastructure is all built out and running, you get into maintenance.

Maintenance, which is by far the longer running part of the lifecycle of the system, consists of really two main "forks" - software upgrades, and hardware failure mitigation.  Of these two, we feel that Spacewalk has a pretty decent handle on software upgrades - it's kind of what RHN was born to do.  Of the latter however - dealing with hardware failures - we run into a conundrum, and whichever scheme is the optimum solution for dealing with hardware failures  very likely plays into whatever the best approach might be for the initial deployment of an infrastructure as well.
## The Conundrum - Prelude



The conundrum lies in workflow complexity and infrastructure requirements.  Here are two scenarios:
### The "start with a working system" approach



This is the approach that spacewalk has taken to date.  In order for a machine to be provisioned, it must first be populated with a very basic "disposable" installation of an operating system capable of initiating a kickstart.  Presumably in this case it makes sense to have either:

    * a collection of machines stashed away that are built out with a "kickstart base" that's compatible with the provisioning infrastructure it's intended to interact with
or
    * a collection of machines of a sort or in a state that they can be quickly provisioned with the most current "kickstart base"

... preparatory work for dealing with machine failures.

Given one of the two above, then, the procedure to replace a machine becomes:

    1. Prep (kickstart) the new machine with a fresh "kickstartable OS" (in the case of the second option above)
    1. Connect the machine to an appropriate spot in the network
    1. Interact with the machine to initiate the kickstart process (e.g. koan or via Spacewalk) such that the correct kickstart profile is selected for building
    1. When the kickstart is complete, rack the machine (if it's not already in place).
### The "start with truly bare metal" approach



This approach presumes that something (e.g. cobbler - or Spacewalk as a front to it) or someone (e.g. operations staff) is coordinating dhcp configuration for the infrastructure in such a way that it is aware of and matched with kickstart profiles defined in spacewalk/cobbler.  The approach to dealing with hardware failure in this case is quite a bit different from the koan/spacewalk approach: all that's really needed (provided that a mechanism for keeping the dhcp <-> kickstart profile tie is in place) is a collection of machines on which there is no operating system, and of which we know only the MAC address of the interface that will attempt a PXE bootstrap when the machine is powered on.

In the event of a hardware failure, then, the procedure to replace a machine becomes:

    1. Record the MAC address of the interface that will attempt a PXE bootstrap when the machine is powered on in spacewalk - replacing the MAC address value of the machine that just failed.
        * This would require either direct (or indirect - through Spacewalk) manipulation of an existing Cobbler system record to update the MAC address field
        * An alternative approach might be to edit an existing system record from a "pool" of bare metal machines such that the appropriate profile and ip address etc. are specified.
    1. Prepare the machine:
        * If the machine is known to have not bootable code on the "first" hard disk, ensure that system BIOS is set to bootstrap in this order:
            a. Hard Disk
            a. PXE
        * If the machine is known to have bootable code on the "first" hard disk, one of two approaches can be taken:
            1. The machine can be booted with sufficient software to wipe any partition tables and boot loaders that might exist on the bootable disk, and then proceed as above or 
            1. The machine's BIOS can be set so that the boot order is:
                a. PXE
                b. Hard Disk
            * In which case it will be necessary to watch for the kickstart process to complete so that the bios settings can be reversed before an "infinite loop of kickstarts" commences.
    1. Connect the machine to the appropriate spot in the network
    1. Power up the machine - the kickstart process will commence automatically due to the fact that the combination of DHCP MAC <--> IP mapping and IP <--> Profile mapping determines the profile.
    1. When the kickstart is complete, rack the machine (if it's not already in place).
## The Conundrum - Analysis



These approaches each have their own advantages and disadvantages.

* The "true bare metal" approach requires a level of integration with the networking infrastructure that's traditionally been squarely in the network administrator's domain, and front end infrastructure configuration requirementsare are complex.

* The "kickstartable base" approach adds overhead to the process in terms maintenance of the collection of "kickstartable base" machines (or the mechanism and procedure used to build them on demand) as it must be ensured that they're compatible with what the provisioning framework expects to interact with (which, itself, changes through time), and there's arguably additional room for human error as the number of non-trivial interactive steps during the process is not small (relatively speaking).
# So....
 


We're thinking this through - we have a lot of options, and so far only our own experience with production deployments to rely on.  What do you think?  Is there some other scenario we're overlooking here - or do you have a preference as to which of these approaches we focus on first?  Please feel free to start or enjoin in a discussion on the spacewalk mailing list.  We have a lot of work ahead of us on this, and we'd like to get the biggest bang for the buck - your input can help us get there faster!