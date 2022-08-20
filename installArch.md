[[Category:Installation process]]
[[bg:Installation guide]]
[[de:Arch Install Scripts]]
[[el:Installation guide]]
[[es:Installation guide]]
[[fa:راهنمای تازه‌کاران]]
[[fi:Installation guide]]
[[fr:Installation guide]]
[[it:Installation guide]]
[[ja:インストールガイド]]
[[ko:Installation guide]]
[[pl:Installation guide]]
[[pt:Installation guide]]
[[ru:Installation guide]]
[[sv:Installation guide]]
[[th:Installation guide]]
[[tr:Installation guide]]
[[uk:Installation guide]]
[[zh-hans:Installation guide]]
[[zh-hant:Installation guide]]
This document is a guide for installing [[Arch Linux]] using the live system booted from an installation medium made from an official installation image. The installation medium provides accessibility features which are described on the page [[Install Arch Linux with accessibility options]]. For alternative means of installation, see [[:Category:Installation process]].

Before installing, it would be advised to view the [[FAQ]]. For conventions used in this document, see [[Help:Reading]]. In particular, code examples may contain placeholders (formatted in {{ic|''italics''}}) that must be replaced manually.

For more detailed instructions, see the respective [[ArchWiki]] articles or the various programs' [[man page]]s, both linked from this guide. For interactive help, the [[IRC channel]] and the [https://bbs.archlinux.org/ forums] are also available.

Arch Linux should run on any [[Wikipedia:X86-64|x86_64]]-compatible machine with a minimum of 512 MiB RAM, though more memory is needed to boot the live system for installation.[https://lists.archlinux.org/archives/list/arch-releng@lists.archlinux.org/message/D5HSGOFTPGYI6IZUEB3ZNAX4D3F3ID37/] A basic installation should take less than 2 GiB of disk space. As the installation process needs to retrieve packages from a remote repository, this guide assumes a working internet connection is available.

== Pre-installation ==

=== Acquire an installation image ===

Visit the [https://archlinux.org/download/ Download] page and, depending on how you want to boot, acquire the ISO file or a netboot image, and the respective [[GnuPG]] signature.

=== Verify signature ===

It is recommended to verify the image signature before use, especially when downloading from an ''HTTP mirror'', where downloads are generally prone to be intercepted to [https://www2.cs.arizona.edu/stork/packagemanagersecurity/attacks-on-package-managers.html serve malicious images]. 

On a system with [[GnuPG]] installed, do this by downloading the ''ISO PGP signature'' ([https://archlinux.org/download/#checksums under Checksums in the page Download]) to the ISO directory, and [[GnuPG#Verify a signature|verifying]] it with: 

 $ gpg --keyserver-options auto-key-retrieve --verify archlinux-''version''-x86_64.iso.sig

Alternatively, from an existing Arch Linux installation run:

 $ pacman-key -v archlinux-''version''-x86_64.iso.sig

{{Note|
* The signature itself could be manipulated if it is downloaded from a mirror site, instead of from [https://archlinux.org/download/ archlinux.org] as above. In this case, ensure that the public key, which is used to decode the signature, is signed by another, trustworthy key. The {{ic|gpg}} command will output the fingerprint of the public key. 
* Another method to verify the authenticity of the signature is to ensure that the public key's fingerprint is identical to the key fingerprint of the [https://archlinux.org/people/developers/ Arch Linux developer] who signed the ISO-file. See [[Wikipedia:Public-key cryptography]] for more information on the public-key process to authenticate keys.
}}

=== Prepare an installation medium ===

The installation image can be supplied to the target machine via a [[USB flash installation medium|USB flash drive]], an [[Optical disc drive#Burning|optical disc]] or a network with [[PXE]]: follow the appropriate article to prepare yourself an installation medium from the chosen image.

=== Boot the live environment ===

{{Note|Arch Linux installation images do not support Secure Boot. You will need to [[Unified Extensible Firmware Interface/Secure Boot#Disabling Secure Boot|disable Secure Boot]] to boot the installation medium. If desired, [[Secure Boot]] can be set up after completing the installation.}}

# Point the current boot device to the one which has the Arch Linux installation medium. Typically it is achieved by pressing a key during the [[Wikipedia:Power-on self test|POST]] phase, as indicated on the splash screen. Refer to your motherboard's manual for details.
# When the installation medium's boot loader menu appears, select ''Arch Linux install medium'' and press {{ic|Enter}} to enter the installation environment. {{Tip|The installation image uses [[GRUB]] for UEFI and [[syslinux]] for BIOS booting. See [https://gitlab.archlinux.org/mkinitcpio/mkinitcpio-archiso/blob/master/docs/README.bootparams README.bootparams] for a list of [[Kernel parameters#Configuration|boot parameters]].}}
# You will be logged in on the first [[Wikipedia:Virtual console|virtual console]] as the root user, and presented with a [[Zsh]] shell prompt.

To switch to a different console—for example, to view this guide with [https://lynx.invisible-island.net/lynx_help/Lynx_users_guide.html Lynx] alongside the installation—use the {{ic|Alt+''arrow''}} [[Keyboard shortcuts|shortcut]]. To [[textedit|edit]] configuration files, {{man|1|mcedit}}, [[nano#Usage|nano]] and [[vim#Usage|vim]] are available. See [https://geo.mirror.pkgbuild.com/iso/latest/arch/pkglist.x86_64.txt pkglist.x86_64.txt] for a list of the packages included in the installation medium.

=== Set the console keyboard layout ===

The default [[console keymap]] is [[Wikipedia:File:KB United States-NoAltGr.svg|US]]. Available layouts can be listed with:

 # ls /usr/share/kbd/keymaps/**/*.map.gz

To set the keyboard layout, pass a corresponding file name to {{man|1|loadkeys}}, omitting path and file extension. For example, to set a [[Wikipedia:File:KB Germany.svg|German]] keyboard layout:

 # loadkeys de-latin1

[[Console fonts]] are located in {{ic|/usr/share/kbd/consolefonts/}} and can likewise be set with {{man|8|setfont}}.

=== Verify the boot mode ===

To verify the boot mode, list the [[efivars]] directory:

 # ls /sys/firmware/efi/efivars

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in [[Wikipedia:BIOS|BIOS]] (or [[Wikipedia:Compatibility Support Module|CSM]]) mode. If the system did not boot in the mode you desired, refer to your motherboard's manual.

=== Connect to the internet ===

To set up a network connection in the live environment, go through the following steps:

* Ensure your [[network interface]] is listed and enabled, for example with {{man|8|ip-link}}: {{bc|# ip link}}
* For wireless and WWAN, make sure the card is not blocked with [[rfkill]].
* Connect to the network:
** Ethernet—plug in the cable.
** Wi-Fi—authenticate to the wireless network using [[iwctl]].
** Mobile broadband modem—connect to the mobile network with the [[mmcli]] utility.
* Configure your network connection:
** [[DHCP]]: dynamic IP address and DNS server assignment (provided by [[systemd-networkd]] and [[systemd-resolved]]) should work out of the box for Ethernet, WLAN, and WWAN network interfaces.
** Static IP address: follow [[Network configuration#Static IP address]].
* The connection may be verified with [[ping]]: {{bc|# ping archlinux.org}}

{{Note|In the installation image, [[systemd-networkd]], [[systemd-resolved]], [[iwd]] and [[ModemManager]] are preconfigured and enabled by default. That will not be the case for the installed system.}}

=== Update the system clock ===

Use {{man|1|timedatectl}} to ensure the system clock is accurate:

 # timedatectl set-ntp true

To check the service status, use {{ic|timedatectl status}}.

=== Partition the disks ===

When recognized by the live system, disks are assigned to a [[block device]] such as {{ic|/dev/sda}}, {{ic|/dev/nvme0n1}} or {{ic|/dev/mmcblk0}}. To identify these devices, use [[lsblk]] or ''fdisk''.

 # fdisk -l

Results ending in {{ic|rom}}, {{ic|loop}} or {{ic|airoot}} may be ignored.

The following [[partition]]s are '''required''' for a chosen device:

* One partition for the [[Wikipedia:Root directory|root directory]] {{ic|/}}.
* For booting in [[UEFI]] mode: an [[EFI system partition]].

If you want to create any stacked block devices for [[LVM]], [[dm-crypt|system encryption]] or [[RAID]], do it now.

Use [[fdisk]] or [[parted]] to modify partition tables. For example:

 # fdisk ''/dev/the_disk_to_be_partitioned''

{{Note|
* If the disk does not show up, [[Partitioning#Drives are not visible when firmware RAID is enabled|make sure the disk controller is not in RAID mode]].
* If the disk from which you want to boot [[EFI system partition#Check for an existing partition|already has an EFI system partition]], do not create another one, but use the existing partition instead.
* [[Swap]] space can be set on a [[swap file]] for file systems supporting it.
}}

==== Example layouts ====

{| class="wikitable"
|+ UEFI with [[GPT]]
|-
! Mount point
! Partition
! [[Wikipedia:GUID Partition Table#Partition type GUIDs|Partition type]]
! Suggested size
|-
| {{ic|/mnt/boot}}<sup>1</sup>
| {{ic|/dev/''efi_system_partition''}}
| [[EFI system partition]]
| At least 300 MiB
|-
| {{ic|[SWAP]}}
| {{ic|/dev/''swap_partition''}}
| Linux swap
| More than 512 MiB
|-
| {{ic|/mnt}}
| {{ic|/dev/''root_partition''}}
| Linux x86-64 root (/)
| Remainder of the device
|}

# [[EFI system partition#Typical mount points|Other mount points]], such as {{ic|/mnt/efi}}, are possible, provided that the used boot loader is capable of loading the kernel and initramfs images from the root volume. See the warning in [[Arch boot process#Boot loader]].

{| class="wikitable"
|+ BIOS with [[MBR]]
|-
! Mount point
! Partition
! [[Wikipedia:Partition type|Partition type]]
! Suggested size
|-
| {{ic|[SWAP]}}
| {{ic|/dev/''swap_partition''}}
| Linux swap
| More than 512 MiB
|-
| {{ic|/mnt}}
| {{ic|/dev/''root_partition''}}
| Linux
| Remainder of the device
|}

See also [[Partitioning#Example layouts]].

=== Format the partitions ===

Once the partitions have been created, each newly created partition must be formatted with an appropriate [[file system]]. See [[File systems#Create a file system]] for details.

For example, to create an Ext4 file system on {{ic|/dev/''root_partition''}}, run:

 # mkfs.ext4 /dev/''root_partition''

If you created a partition for [[swap]], initialize it with {{man|8|mkswap}}:

 # mkswap /dev/''swap_partition''

{{Note|For stacked block devices replace {{ic|/dev/''*_partition''}} with the appropriate block device path.}}

If you created an EFI system partition, [[EFI system partition#Format the partition|format it]] to FAT32 using {{man|8|mkfs.fat}}.

{{Warning|Only format the EFI system partition if you created it during the partitioning step. If there already was an EFI system partition on disk beforehand, reformatting it can destroy the boot loaders of other installed operating systems.}}

 # mkfs.fat -F 32 /dev/''efi_system_partition''

=== Mount the file systems ===

[[Mount]] the root volume to {{ic|/mnt}}. For example, if the root volume is {{ic|/dev/''root_partition''}}:

 # mount /dev/''root_partition'' /mnt

Create any remaining mount points (such as {{ic|/mnt/efi}}) and mount their corresponding volumes.

{{Tip|Run {{man|8|mount}} with the {{ic|--mkdir}} option to create the specified mount point. Alternatively, create it using {{man|1|mkdir}} beforehand.}}

For UEFI systems, mount the EFI system partition:

 # mount --mkdir /dev/''efi_system_partition'' /mnt/boot

If you created a [[swap]] volume, enable it with {{man|8|swapon}}:

 # swapon /dev/''swap_partition''

{{man|8|genfstab}} will later detect mounted file systems and swap space.

== Installation ==

=== Select the mirrors ===

Packages to be installed must be downloaded from [[Mirrors|mirror servers]], which are defined in {{ic|/etc/pacman.d/mirrorlist}}. On the live system, after connecting to the internet, [[reflector]] updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, [[textedit|edit]] the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by ''pacstrap'', so it is worth getting right.

=== Install essential packages ===

Use the {{man|8|pacstrap}} script to install the {{Pkg|base}} package, Linux [[kernel]] and firmware for common hardware:

 # pacstrap /mnt base linux linux-firmware

{{Tip|
* You can substitute {{Pkg|linux}} for a [[kernel]] package of your choice, or you could omit it entirely when installing in a [[Wikipedia:Container (virtualization)|container]].
* You could omit the installation of the firmware package when installing in a virtual machine or container.
}}

The {{Pkg|base}} package does not include all tools from the live installation, so installing other packages may be necessary for a fully functional base system. In particular, consider installing:

* userspace utilities for the management of [[file systems]] that will be used on the system,
* utilities for accessing [[RAID]] or [[LVM]] partitions,
* specific firmware for other devices not included in {{Pkg|linux-firmware}} (e.g. {{Pkg|sof-firmware}} for [[Advanced Linux Sound Architecture#ALSA firmware|sound cards]]),
* software necessary for [[networking]] (e.g. a network manager or DHCP client),
* a [[text editor]],
* packages for accessing documentation in [[man]] and [[info]] pages: {{Pkg|man-db}}, {{Pkg|man-pages}} and {{Pkg|texinfo}}.

To [[install]] other packages or package groups, append the names to the ''pacstrap'' command above (space separated) or use [[pacman]] while [[#Chroot|chrooted into the new system]]. For comparison, packages available in the live system can be found in [https://geo.mirror.pkgbuild.com/iso/latest/arch/pkglist.x86_64.txt pkglist.x86_64.txt].

== Configure the system ==

=== Fstab ===

Generate an [[fstab]] file (use {{ic|-U}} or {{ic|-L}} to define by [[UUID]] or labels, respectively):

 # genfstab -U /mnt >> /mnt/etc/fstab

Check the resulting {{ic|/mnt/etc/fstab}} file, and [[textedit|edit]] it in case of errors.

=== Chroot ===

[[Change root]] into the new system:

 # arch-chroot /mnt

=== Time zone ===

Set the [[time zone]]:

 # ln -sf /usr/share/zoneinfo/''Region''/''City'' /etc/localtime

Run {{man|8|hwclock}} to generate {{ic|/etc/adjtime}}:

 # hwclock --systohc

This command assumes the hardware clock is set to [[Wikipedia:UTC|UTC]]. See [[System time#Time standard]] for details.

=== Localization ===

[[textedit|Edit]] {{ic|/etc/locale.gen}} and uncomment {{ic|en_US.UTF-8 UTF-8}} and other needed [[locale]]s. Generate the locales by running:

 # locale-gen

[[Create]] the {{man|5|locale.conf}} file, and [[Locale#Setting the system locale|set the LANG variable]] accordingly:

{{hc|1=/etc/locale.conf|2=
LANG=''en_US.UTF-8''
}}

If you [[#Set the console keyboard layout|set the console keyboard layout]], make the changes persistent in {{man|5|vconsole.conf}}:

{{hc|1=/etc/vconsole.conf|2=
KEYMAP=''de-latin1''
}}

=== Network configuration ===

[[Create]] the [[hostname]] file:

{{hc|/etc/hostname|
''myhostname''
}}

Complete the [[network configuration]] for the newly installed environment. That may include installing suitable [[network management]] software.

=== Initramfs ===

Creating a new ''initramfs'' is usually not required, because [[mkinitcpio]] was run on installation of the [[kernel]] package with ''pacstrap''.

For [[Install Arch Linux on LVM#Adding mkinitcpio hooks|LVM]], [[dm-crypt|system encryption]] or [[RAID#Configure mkinitcpio|RAID]], modify {{man|5|mkinitcpio.conf}} and recreate the initramfs image:

 # mkinitcpio -P

=== Root password ===

Set the root [[password]]:

 # passwd

=== Boot loader ===

Choose and install a Linux-capable [[boot loader]]. If you have an Intel or AMD CPU, enable [[microcode]] updates in addition.

== Reboot ==

Exit the chroot environment by typing {{ic|exit}} or pressing {{ic|Ctrl+d}}.

Optionally manually unmount all the partitions with {{ic|umount -R /mnt}}: this allows noticing any "busy" partitions, and finding the cause with {{man|1|fuser}}.

Finally, restart the machine by typing {{ic|reboot}}: any partitions still mounted will be automatically unmounted by ''systemd''. Remember to remove the installation medium and then login into the new system with the root account.

== Post-installation ==

See [[General recommendations]] for system management directions and post-installation tutorials (like creating unprivileged user accounts, setting up a graphical user interface, sound or a touchpad).

For a list of applications that may be of interest, see [[List of applications]].
