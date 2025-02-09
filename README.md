## Summary
This is a proof-of-concept of a headless Void Linux setup which was configured to be extremely lightweight. It's configured to have no video, thus being truly headless whilst using serial port interface. My current setup fits 256MB of VM RAM and 8GB of VM disk (of which only 3GB are in use).

## ... no video?!
Exactly, no video, literally! The VM is configured without graphics adapters. There's no SSH daemon either. The serial port (ttyS0) is the only interface to the machine. I chose not to use SSH because serial comm is something well-engrained within Linux distros (also because I'm intending to use it as a very-tiny firewall to another VM). 

Both this README and this repo are being made while I'm accessing the serial interface (a setup which involves the use of socat and GNU screen from my host machine to my guest's serial port being served by VirtualBox through localhost TCP).

## Purpose
"Just because it can be done". 
(Also, I git-ted the root of this Linux filesystem, as you can see by my .gitignore, "just because it can be done")

## Currently unsolvable issues
- VirtualBox doesn't accept exactly 0MB of video memory, even when graphics controller is set to "None". Something about "framebuffer".
- The "btrfs" can't get ridden of. It's probably a hard dependency. 
- initramfs can't get smaller without starting to break things. 47MB is the smallest I could get it to be (which is still impressive for a distro that requires at least 2GB to boot without kernel panics, now booting with 256MB without a hassle).

## Current setup performance and settings
### RAM 
(Injected to this README via `:r!free -h`, unedited)

```
               total        used        free      shared  buff/cache   available
Mem:           197Mi        49Mi        42Mi       380Ki       112Mi       147Mi
Swap:          499Mi          0B       499Mi
```

(the virtual machine is configured with 256MB, so there's almost 60MB missing from total returned by `free -h`)

### Disk 
(Injected to this README via `:r!du -hd 1 / | sort -h`, manually edited to remove the error regarding `/proc/something/` that despawned during the calculation of `du`)

```
0	./dev
0	./proc
0	./sys
0	./tmp
4.0K	./media
4.0K	./mnt
4.0K	./opt
16K	./lost+found
32K	./home
192K	./.git
348K	./root
380K	./run
13M	./etc
203M	./boot
618M	./var
1.5G	./usr
2.3G	.
```

### Current packages explicitly installed (XBPS Package Manager)
(Injected to this README via `:r!xbps-query -m`, unedited)

```
base-system-0.114_2
git-2.48.1_1
grml-zsh-config-0.19.4_1
grub-2.12_2
grub-x86_64-efi-2.12_2
htop-3.3.0_1
tcsh-6.24.15_1
vbetool-1.1_5
vim-9.1.0772_2
zsh-5.9_3
```

### Currently running processes
(Injected to this README via `:r!ps -aux`, manually edited to remove kernel threads, and to remove the long `current log` thing from my previous git commit)  

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   1004   100 ?        Ss   18:38   0:00 runit
root       526  0.0  0.5   2540  1152 ?        Ss   18:38   0:00 runsvdir -P /run/runit/runsvdir/current log: ...
root       532  0.2  0.6   2388  1232 ?        Ss   18:38   0:15 runsv ip6tables
root       533  0.0  0.6   2388  1264 ?        Ss   18:38   0:00 runsv agetty-tty1
root       534  0.0  0.6   2388  1228 ?        Ss   18:38   0:00 runsv udevd
root       535  0.0  0.5   2388  1192 ?        Ss   18:38   0:00 runsv dhcpcd-eth0
root       536  0.1  0.6   2388  1232 ?        Ss   18:38   0:13 runsv iptables
root       537  0.0  0.6   2388  1244 ?        Ss   18:38   0:00 runsv agetty-ttyS0
root       538  0.0  0.9   6072  1996 tty1     Ss+  18:38   0:00 agetty --noclear tty1 38400 linux
root       539  0.0  0.5   2508  1108 ?        S    18:38   0:00 vlogger -t ip6tables -p daemon
root       540  0.0  0.7   2508  1548 ?        S    18:38   0:00 vlogger -t dhcpcd-eth0 -p daemon
root       542  0.0  1.0   2964  2032 ?        S    18:38   0:00 dhcpcd: enp0s3 [ip4] [ip6]
root       543  0.0  0.5   2508  1108 ?        S    18:38   0:00 vlogger -t udevd -p daemon
root       544  0.0  1.1  17396  2376 ?        S    18:38   0:00 udevd
root       546  0.0  0.6   2508  1236 ?        S    18:38   0:00 vlogger -t iptables -p daemon
root       899  0.3  4.9  14400  9940 ttyS0    Sl+  20:35   0:00 vim README.md
root      2247  0.0  1.0   3260  2044 ?        Ss   18:39   0:00 login -- root
root      2368  0.0  1.8   7388  3812 ttyS0    Ss   18:39   0:00 -bash
root      6791 50.0  1.6   7040  3372 ttyS0    S+   20:39   0:00 /bin/bash -c (ps -aux)>/tmp/vytvA0f/3 2>&1
root      6792  0.0  1.7   7200  3484 ttyS0    R+   20:39   0:00 ps -aux
```

### Current cmdline
(Injected to this README via `:r!cat /proc/cmdline`, manually edited to redact my filesystem's UUID)

```
BOOT_IMAGE=/boot/vmlinuz-6.12.12_1 root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx ro loglevel=4 console=ttyS0
```

### Current VM settings
(Pasted from the host's `VBoxManage showvminfo name-of-this-vm`, manually edited to redact some things and to emphasize other things)

```
Name:                        (doesn't matter)
Groups:                      /
Guest OS:                    Linux 2.6 / 3.x / 4.x (64-bit)
...
Memory size:                 256MB
Page Fusion:                 disabled
VRAM size:                   1MB
CPU exec cap:                100%
HPET:                        disabled
CPUProfile:                  host
Chipset:                     piix3
Firmware:                    EFI
Number of CPUs:              2
PAE:                         disabled
Long Mode:                   enabled
Triple Fault Reset:          disabled
APIC:                        enabled
X2APIC:                      enabled
Nested VT-x/AMD-V:           disabled
CPUID Portability Level:     0
CPUID overrides:             None
Boot menu mode:              message and menu
Boot Device 1:               DVD
Boot Device 2:               HardDisk
Boot Device 3:               Not Assigned
Boot Device 4:               Not Assigned
ACPI:                        enabled
IOAPIC:                      enabled
BIOS APIC mode:              APIC
Time offset:                 0ms
BIOS NVRAM File:             (doesn't matter)
RTC:                         UTC
Hardware Virtualization:     enabled
Nested Paging:               enabled
Large Pages:                 disabled
VT-x VPID:                   enabled
VT-x Unrestricted Exec.:     enabled
Paravirt. Provider:          Default
Effective Paravirt. Prov.:   KVM
State:                       running (since 2025-02-08T21:38:12.755000000)
Graphics Controller:         Null (***<-- Notice this***)
Monitor count:               1
3D Acceleration:             disabled
2D Video Acceleration:       disabled
Teleporter Enabled:          disabled
Teleporter Port:             0
Teleporter Address:          
Teleporter Password:         
Tracing Enabled:             disabled
Allow Tracing to Access VM:  disabled
Tracing Configuration:       
Autostart Enabled:           disabled
Autostart Delay:             0
Default Frontend:            
VM process priority:         default
Storage Controller Name (0):            AHCI
Storage Controller Type (0):            IntelAhci
Storage Controller Instance Number (0): 0
Storage Controller Max Port Count (0):  30
Storage Controller Port Count (0):      2
Storage Controller Bootable (0):        on
Storage Controller Name (1):            USB
Storage Controller Type (1):            USB
Storage Controller Instance Number (1): 0
Storage Controller Max Port Count (1):  8
Storage Controller Port Count (1):      8
Storage Controller Bootable (1):        on
AHCI (0, 0): Empty
AHCI (1, 0): /the-disk-drive-which-doesnt-matter.vdi (UUID: ...)
NIC 1:                       MAC: xxxxxxxxxxxx, Attachment: NAT, Cable connected: on, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 1 Settings:  MTU: 0, Socket (send: 64, receive: 64), TCP Window (send:64, receive: 64)
NIC 2:                       MAC: xxxxxxxxxxxx, Attachment: Internal Network '...', Cable connected: off, Trace: off (file: none), Type: 82540EM, Reported speed: 0 Mbps, Boot priority: 0, Promisc Policy: deny, Bandwidth group: none
NIC 3:                       disabled
NIC 4:                       disabled
NIC 5:                       disabled
NIC 6:                       disabled
NIC 7:                       disabled
NIC 8:                       disabled
Pointing Device:             PS/2 Mouse
Keyboard Device:             PS/2 Keyboard
UART 1:                      I/O base: 0x03f8, IRQ: 4, attached to tcp (server) 'xxxx', 16550A
UART 2:                      disabled
UART 3:                      disabled
UART 4:                      disabled
LPT 1:                       disabled
LPT 2:                       disabled
Audio:                       disabled
Audio playback:              disabled
Audio capture:               disabled
Clipboard Mode:              disabled
Drag and drop Mode:          disabled
Session name:                headless
Video mode:                  0x0x0 at 0,0 enabled
VRDE:                        disabled
OHCI USB:                    enabled
EHCI USB:                    enabled
xHCI USB:                    disabled
```

