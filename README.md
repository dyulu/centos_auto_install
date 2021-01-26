
---

Host kickstart file on a http server:
http://1.2.3.4/centos_8_boot_iso_custom_ks.cfg

---

CentOS-8.2.2004-x86_64-boot.iso from https://www.centos.org/download/ contains only the installer, but not any installable packages.
Modify CentOS-8.2.2004-x86_64-boot.iso boot option to boot using the kickstart file:

PWD: centos8/
CentOS-8.2.2004-x86_64-boot.iso
centos8/iso_mnt
centos8/iso_custom

mount -t iso9660 -o loop CentOS-8.2.2004-x86_64-boot.iso iso_mnt
shopt -s dotglob
cp -avRf iso_mnt/* iso_custom
umount iso_mnt
Edit iso_custom/isolinux/isolinux.cfg to default to kickstart method for installation:
    label linux
      menu label ^Install CentOS Linux 8
      menu default
      kernel vmlinuz
      append initrd=initrd.img inst.ks=http://1.2.3.4/centos_8_boot_iso_custom_ks.cfg
mkisofs -J -T -o CentOS-8.2.2004-x86_64-custom-boot.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -m TRANS.TBL -graft-point -V “CentOS-8.2” iso_custom

---

label linux
  menu label ^Install CentOS Linux 8
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS-8-2-2004-x86_64-dvd inst.ks=http://1.2.3.4/centos_8_minimal_iso_custom_ks.cfg
  
mkisofs -J -T -o CentOS-8.2.2004-x86_64-custom-minimal.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -m TRANS.TBL -graft-point -V “CentOS-8-2-2004-x86_64-dvd” iso_custom
  
Note: 
1. The iso volume ID has to match the LABEL used in isolinux.cfg
2. To enable monitoring install progress via BMC SOL, add kernel option: console=ttyS1,115200n8
3. Make sure file with name .treeinfo exist
$ find . -name .treeinfo
./iso_custom/.treeinfo

inst.stage2=
    Specifies the location of the installation program runtime image to be loaded. The syntax is the same as inst.repo. This option expects a path to a directory containing a valid .treeinfo file; the location of the runtime image will be read from this file if found. If a .treeinfo file is not available, Anaconda will try to load the image from LiveOS/squashfs.img. 

inst.repo=
    Specifies the installation source - that is, a location where the installation program can find the images and packages it requires, e.g., inst.repo=cdrom
    
console=
    This kernel option specifies a device to be used as the primary console. For example, to use a console on the first serial port, use console=ttyS0. This option should be used along with the inst.text option.
    You can use this option multiple times. In that case, the boot message will be displayed on all specified consoles, but only the last one will be used by the installation program afterwards. For example, if you specify console=ttyS0 console=ttyS1, the installation program will use ttyS1. 

---

iso_mnt
├── EFI
│   └── BOOT
│       ├── BOOT.conf
│       ├── BOOTIA32.EFI
│       ├── BOOTX64.EFI
│       ├── fonts
│       │   ├── TRANS.TBL
│       │   └── unicode.pf2
│       ├── grub.cfg
│       ├── grubia32.efi
│       ├── grubx64.efi
│       ├── mmia32.efi
│       ├── mmx64.efi
│       └── TRANS.TBL
├── images
│   ├── efiboot.img
│   ├── install.img
│   ├── pxeboot
│   │   ├── initrd.img
│   │   ├── TRANS.TBL
│   │   └── vmlinuz
│   └── TRANS.TBL
└── isolinux
    ├── boot.cat
    ├── boot.msg
    ├── grub.conf
    ├── initrd.img
    ├── isolinux.bin
    ├── isolinux.cfg
    ├── ldlinux.c32
    ├── libcom32.c32
    ├── libutil.c32
    ├── memtest
    ├── splash.png
    ├── TRANS.TBL
    ├── vesamenu.c32
    └── vmlinuz

6 directories, 31 files

$ isoinfo -d -i CentOS-8.2.2004-x86_64-boot.iso
CD-ROM is in ISO 9660 format
System id: LINUX
Volume id: CentOS-8-2-2004-x86_64-dvd
Volume set id: 
Publisher id: 
Data preparer id: 
Application id: GENISOIMAGE ISO 9660/HFS FILESYSTEM CREATOR (C) 1993 E.YOUNGDALE (C) 1997-2006 J.PEARSON/J.SCHILLING (C) 2006-2007 CDRKIT TEAM
Copyright File id: 
Abstract File id: 
Bibliographic File id: 
Volume set size is: 1
Volume set sequence number is: 1
Logical block size is: 2048
Volume size is: 319029
El Torito VD version 1 found, boot catalog is in sector 44
Joliet with UCS level 3 found
Rock Ridge signatures version 1 found
Eltorito validation header:
    Hid 1
    Arch 0 (x86)
    ID ''
    Key 55 AA
    Eltorito defaultboot header:
        Bootid 88 (bootable)
        Boot media 0 (No Emulation Boot)
        Load segment 0
        Sys type 0
        Nsect 4
        Bootoff 4C99A 313754

$ isoinfo -p -i CentOS-8.2.2004-x86_64-boot.iso
Path table starts at block 21, size 94
   1:    1 1d 
   2:    1 21 EFI
   3:    1 1e IMAGES
   4:    1 20 ISOLINUX
   5:    2 22 BOOT
   6:    3 1f PXEBOOT
   7:    5 23 FONTS

$ isoinfo -l -i CentOS-8.2.2004-x86_64-boot.iso

Directory listing of /
d---------   0    0    0            2048 Jun  8 2020 [     29 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     29 02]  .. 
d---------   0    0    0            2048 Jun  8 2020 [     33 02]  EFI 
d---------   0    0    0            2048 Jun  8 2020 [     30 02]  IMAGES 
d---------   0    0    0            2048 Jun  8 2020 [     32 02]  ISOLINUX 

Directory listing of /EFI/
d---------   0    0    0            2048 Jun  8 2020 [     33 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     29 02]  .. 
d---------   0    0    0            2048 Jun  8 2020 [     34 02]  BOOT 

Directory listing of /IMAGES/
d---------   0    0    0            2048 Jun  8 2020 [     30 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     29 02]  .. 
----------   0    0    0        10434560 Jun  8 2020 [     45 00]  EFIBOOT.IMG;1 
----------   0    0    0       558002176 Jun  8 2020 [   5140 00]  INSTALL.IMG;1 
d---------   0    0    0            2048 Jun  8 2020 [     31 02]  PXEBOOT 
----------   0    0    0             446 Jun  8 2020 [ 277602 00]  TRANS.TBL;1 

Directory listing of /ISOLINUX/
d---------   0    0    0            2048 Jun  8 2020 [     32 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     29 02]  .. 
----------   0    0    0            2048 Jun  8 2020 [     44 00]  BOOT.CAT;1 
----------   0    0    0              84 Jun  8 2020 [ 313752 00]  BOOT.MSG;1 
----------   0    0    0             293 Jun  8 2020 [ 313753 00]  GRUB.CON;1 
----------   0    0    0        65114456 Jun  8 2020 [ 277603 00]  INITRD.IMG;1 
----------   0    0    0           38912 Nov  8 2019 [ 313754 00]  ISOLINUX.BIN;1 
----------   0    0    0            3075 Jun  8 2020 [ 313773 00]  ISOLINUX.CFG;1 
----------   0    0    0          116096 Nov  8 2019 [ 313775 00]  LDLINUX.C32;1 
----------   0    0    0          180700 Nov  8 2019 [ 313832 00]  LIBCOM32.C32;1 
----------   0    0    0           22804 Nov  8 2019 [ 313921 00]  LIBUTIL.C32;1 
----------   0    0    0          182704 May 13 2019 [ 313933 00]  MEMTEST.;1 
----------   0    0    0             186 Jul 30 2019 [ 314023 00]  SPLASH.PNG;1 
----------   0    0    0            2885 Jun  8 2020 [ 314024 00]  TRANS.TBL;1 
----------   0    0    0           26788 Nov  8 2019 [ 314026 00]  VESAMENU.C32;1 
----------   0    0    0         8913656 May  8 2020 [ 309399 00]  VMLINUZ.;1 

Directory listing of /EFI/BOOT/
d---------   0    0    0            2048 Jun  8 2020 [     34 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     33 02]  .. 
----------   0    0    0            1362 Jun  8 2020 [ 314040 00]  BOOT.CON;1 
----------   0    0    0          975688 May  7 2020 [ 314041 00]  BOOTIA32.EFI;1 
----------   0    0    0         1211224 May  7 2020 [ 314518 00]  BOOTX64.EFI;1 
d---------   0    0    0            2048 Jun  8 2020 [     35 02]  FONTS 
----------   0    0    0            1362 Jun  8 2020 [ 315110 00]  GRUB.CFG;1 
----------   0    0    0         1181576 Apr 14 2020 [ 315111 00]  GRUBIA32.EFI;1 
----------   0    0    0         1877384 Apr 14 2020 [ 315688 00]  GRUBX64.EFI;1 
----------   0    0    0          927888 May  7 2020 [ 316605 00]  MMIA32.EFI;1 
----------   0    0    0         1160136 May  7 2020 [ 317059 00]  MMX64.EFI;1 
----------   0    0    0            1995 Jun  8 2020 [ 317626 00]  TRANS.TBL;1 

Directory listing of /IMAGES/PXEBOOT/
d---------   0    0    0            2048 Jun  8 2020 [     31 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     30 02]  .. 
----------   0    0    0        65114456 Jun  8 2020 [ 277603 00]  INITRD.IMG;1 
----------   0    0    0             441 Jun  8 2020 [ 309398 00]  TRANS.TBL;1 
----------   0    0    0         8913656 May  8 2020 [ 309399 00]  VMLINUZ.;1 

Directory listing of /EFI/BOOT/FONTS/
d---------   0    0    0            2048 Jun  8 2020 [     35 02]  . 
d---------   0    0    0            2048 Jun  8 2020 [     34 02]  .. 
----------   0    0    0             223 Jun  8 2020 [ 317627 00]  TRANS.TBL;1 
----------   0    0    0         2560080 Apr 14 2020 [ 317628 00]  UNICODE.PF2;1 


