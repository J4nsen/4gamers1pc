# 4gamers1pc
![4Gamers1PC](https://imgur.com/WeOE5yj.jpg)

Hey, in this project I want to present my "4 Gamers, 1 PC"-setup (more pictures at the bottom).
It is basically just a PC with two CPUs, support for four GPUs and libvirt+KVM+Qemu. 
The selected hardware works without any quirks, which is the reason why I
want to share my setup.

My friends and I like to meet in our home town
once a month and game together. A small LAN-party. The plan for this build was that they just 
bring their own GPU, we put it in the gaming server and are ready to go.

My secondary goals for this project are having a beefy workstation and an always up-to-date PC with minimal 
maintenance. Side effects are no broken hard-drives, no rattling CPU fans, no outdated games, 
no carrying PCs through half of the country for a gaming session, and of course: Linux rocks! 
If I want to do some work on this machine, I boot a second Linux with no VM stuff installed. 
I dont use any fancy GPU detach/attach scripts.

If you have questions just open an issue, I will continue to update this readme.

## Introduction
I always liked the idea of using the power of my PC to the maximum. My first 
steps were turning my server in the cellar into a terminal server with the
help of the Linux Terminal Server Project. However this was just suiteable for 
non 3D-Work.
### First Try with Xen
Many years later open source virtualization with XEN caught my attention and its
feature to passthrough a GPU to a virtual machine. After some patching, I 
managed to passthough the IGPU of my Intel i5-2500. I was super happy to be able
to play Counter-Strike in a VM, even with no sound. Sadly, I could not manage 
to forward more powerful GPUs like my Nvidia 680. There were some patches on the
mailing list, but I just could not get it to work.
### KVM to the Rescue
Fast foward to end of 2015. The whole hardware-passthrough situation got a lot better. 
After watching Linus "7 Gamers 1 CPU"-video [1] I was hooked again and tried to 
setup my own "2 Gamers 1 CPU" (still using my i5-2500). 
It worked without too much fiddeling. The blog posts by Alex Williamson were a 
great help [2] and I was able to passthrough my two Nvidia GPUs. However, the 
performance in Heroes of the Storm wasn't great. First, because I didn't do any
tuning (CPU pinning etc) and secondly, HotS does not run well on Windows 10 
(what I did not know at this time).
### Final Version
In Mid 2016 a post on the vfio mailinglist caught my attention: 
Cheap (100€) Sandybridge Xeons with 8 cores and good vfio support. 
With two of them I could build my own "4 Gamers, 1 PC"-setup. I could not resist anymore...

## Hardware
* **Mainboard:** ASRock Rack EP2C602 [320€]
  * It has enough PCI-E Slots for 4 GPUs and two USB3 controllers and built-in GPU. Additionally the two USB2 controllers in the chipset can individually be assigned to VMs. The host can still be controlled by a PS2 keyboard. IPMI!
* **CPUs:** 2x Intel E5-2670 [200€]
  * **Cheap**. ~100€ per CPU on eBay.
  * Turboboosts up to 3.3 GHz. ACS support. 40 PCIe lanes. 
  * I think this CPU will still provide enough power in the future, when more games will use DX12/Vulkan, which can send drawcalls from different threads/CPUs and thus making a high clocking CPU core less important.
* **RAM:** 64 GB of Hynix HMT31GR7BFR4C-H9 8GB DIMM DDR3 1333 MHz [200€]
  * ECC RAM
  * Much cheaper than non ECC-RAM and not slower. [8]
* **Cooler:** 2x Corsair CW-9060013-WW Hydro Series [180€]
  * Started with two Scythe Mugen 3, but switched to AIO watercooling, because the Mugens did not fit correctly. Furthermore I do not want to put too much stress on the mainboard.
* **GPUs:** (Nvidia) GPUs with UEFI Support
  * I personally own a GeForce 1070 (MSI) and 970 (Palit JetStream).
  * UEFI support in the GPU's BIOS makes the setup much more easy and stable.
* **USB:** 2x USB3 PCI-e controllers with Renesas chipset. [30€]
  * I've tested one controller with a VIA chipset. I didn't work well, e.g., I wasn't able to move the mouse cursor after a while. No error messages in the VM or host.
  * *Alternative*: Sonnet Allegro Pro USB 3.0 PCIe card [6]. It has four dedicated USB host controllers connected to a PCIe switch and proper ACS support [7]. So one port/host-controller for each VM! 
* **Sound**: USB soundcard
  * I did not want to deal with emulated soundcards, since I've read a lot about crackling, stutters etc (Should be much better now [9]).
  * I personally use a DAC + AMP to drive two loudspeakers (SMSL Q5 50W pro [100€]).
* **Case:** Corsair Obsidian 750D [130€]
  * Enough space for an E-ATX mainboard and AIO water-cooling
* **Powersupply:** Corsair RMx Series RM1000x 1000W [160€]
  * Can power four GPUs and two CPUs. 10 years warranty (at least in Germany).
* **GPU-Case:** *Grakomat*. Custom-built
  * The case/mainboard configuration cannot hold four GPUs and two USB 3 Controllers. Hence, I've built (with a friend of mine) a custom solution, called *Grakomat*.
* **PCIe Extenders:** 4x 50cm of shielded PCI-E Riser Cables by 3M [3] [400€]
  * I did not even try to use cheap unshielded ones. At the time of building the PC, these were the only ones available. Today, there are more [4].

About 1600€ (without GPUs/Soundcard/Grakomat). Can be done much cheaper: Air cooling (~-100€), no Grakomat (=> Only three GPUs) (-400€) => 1100€
### Grakomat
Four GPUs do not fit in the case, nor on the mainboard 
(except if you use single-slotted GPUs). Three GPUs work, however adequate 
cooling is problematic. Luckily, a friend of mine is good in CAD and has access to a lot of machinery. 
After some hours in Inventor (Autodesk) and measuring the case, 
we had a prototype. Fortunately, we found most 3D models online (Riser cable, GPUs), 
which made designing a lot easier. The files are located in the *cad/* directory 
(Autodesk Inventor files; German; Sorry).

## Software
* **Host Setup**: Arch Linux
  * I just followed the Arch Wiki. [5]
* **CPUs + NUMA**
  * Since the host has two CPU sockets, we have to deal with NUMA. A VM running on a CPU core in socket #1 and using RAM of the CPU in Socket #2 would result in not optimal performance. Just like that, a VM running on a CPU in Socket #1, but using a GPU, which is (electrically) connected through its PCI-e lanes to the CPU in Socket #2 will result in bad performance. The VM configs take care of that.
* **VM libvirt configs**
  * Contains: CPU/Emulator Pinning, NUMA-aware memory allocation, Nvidia Error 43 workaround, 
  * Located in *configs/*
* **LVM Setup**
  * Each VM has a persistent logical volume with Windows 10/8 and Linux. When a 'slave'-VM starts, a script is executed which creates a snapshot of the logical volume which saves the games. This also works for the OS part, but makes the startup a bit slower, since Windows needs to reconfigure the whole hardware. It also makes new users or savegame states volatile. 
* **Scripts**
  * Just one script + config file, which gets executed when a VM starts or quits. It creates and deletes LV-snapshots of the gaming-LV. 
  * Located in *scripts/*
* **Windows**
  * Enable MS-Interrupts (MSI) for the GPUs and USB Controllers. This *should* result in better performance, but I haven't benchmarked it, yet.

## Bugs/Problems
* In the beginning I used the outer most PCI-E x4 slot (white color) with a GPU and saw some PCI-E link errors in the hosts log. They often resulted in a freeze of the VM. Sometimes Windows was able to reset the GPU driver and recover. Setting this port to PCI-E Gen 2 (from 3) in the UEFI fixed it. In the final setup a USB controller populates this port with no problems on Gen 3 speeds.
* Some problem with an older GPU (EVGA GeForce GTX 660). The Kernel shows a lot of "PCIe Bus Error" (Bad TLP) messages. They did not result in any crashes, since the systen was able to correct them ("severity=Corrected"). Setting the PCIe generation to 2.0 fixes the problem for this specific card.
* Some games/anti cheat engines detect that they are running in a VM and quit (e.g. CS:GO).
* Heroes of the Storm is running bad on Windows 10 [10]. Rebooting to Windows 8 fixes that.
* My newer Dell monitors sometimes flicker/go black for a short period of time. However, I think this has nothing to do with the VM setup, but with the length of the Displayport cables (4 meters). 
* Too much warm air under the table. ;)

## Todo
* Test hugepages

## More Pictures
I've created an imgur album: https://imgur.com/a/Husi2

## Sources
* [1] https://www.youtube.com/watch?v=LXOaCkbt4lI
* [2] http://vfio.blogspot.de/2015/05/vfio-gpu-how-to-series-part-1-hardware.html
* [3] https://www.digikey.com/products/en?mpart=8KC3-0726-0500&v=19
* [4] https://videocardz.com/review/pci-express-riser-extender-test
* [5] https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
* [6] https://www.amazon.com/Sonnet-Allegro-Pro-PCIe-card/dp/B00XPUHO10/
* [7] https://www.redhat.com/archives/vfio-users/2017-November/msg00017.html
* [8] https://www.pugetsystems.com/labs/articles/ECC-and-REG-ECC-Memory-Performance-560/
* [9] https://www.reddit.com/r/VFIO/comments/74vokw/improved_pulse_audio_driver_for_qemu/
* [10] https://www.redhat.com/archives/vfio-users/2017-January/msg00012.html
