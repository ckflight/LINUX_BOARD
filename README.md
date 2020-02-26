# LINUX_BOARD
AT91SAM9N12 based embedded linux board hardware and operating system files

![IMG_20200227_002110](https://user-images.githubusercontent.com/61315249/75391102-8bdff500-58fa-11ea-9e2c-1d8b6b9425e5.jpg)

![Untitled](https://user-images.githubusercontent.com/61315249/75203603-9b86fe80-577f-11ea-85d4-b7b913c84fa1.png)


  STEP BY STEP GUIDE FOR FLASHING BOARD TO RUN LINUX
  TESTED WITH UBUNTU 16.04.6 and 18.04.2 LTS
  
  Both uboot and at91bootstrap configured to 115200 8N1 no parity. 
  When buildroot starts (linux itself), it also starts uart with 115200. 

  IMPORTANT: Do not use screen terminal command to monitor it does not show correct even baud rate set correctly.
             Use TeraTerm on Windows or Picocom on Mac and set baud rate to 118200 8bit no parity 1 stop it works perfectly.
	     In at91bootstrap mcu clock is configured from 400MHz to 300MHz for changing RAM from 133MHz to 100MHz.
	     but speed is 301.333 MHz and 101.333 so baudrate is not working correctly 115200 that is why baudrate is changed 		   to 118200.


1. This part describes how board is connected over USB and what commands needed to see on sam_ba program

   If linux board is new and fresh then connect over usb to linux virtual computer
   If linux board is flashed previously with files then connect nandflash's pin41 to ground and plug usb
 
   Linux will see device as /dev/ttyACM0 but sam_ba cannot see so type command (It is required every time):
 	
	sudo chmod a+rw /dev/ttyACM0 
   or 

   Add read write permission to usb device to not write above command every time.
   Go to /etc/udev/rules.d and create a rule file such as myusb.rules file and add the line
 	
	sudo nano /etc/udev/rules.d/filename.rules
	KERNEL=="ttyACM[0-9]*",MODE="0666"

   Now sam-ba can see board as /dev/ttyACM0


2. Before start install these libraries:

	1. sudo apt install libncurses5 libncursesw5-dev libncurses5-dev
	2. sudo apt install make
	3. sudo apt install gcc-arm-linux-gnueabi
	4. sudo apt install gcc
	5. sudo apt install g++
	6. sudo apt install python
        (these might no be needed)
	7. sudo apt install make-guile
	8. sudo apt install gcc-multilib
	9. sudo apt install gcc-arm*

	Files required for board and in this part they will be created:
	A. AT91SAMBOOTSTRAP 	(Initialises SAM9N12E hardware like clock, nano interface scram gpio etc.)
	B. U-BOOT    		(Initialises SAM9N12E hardware clk, ram and etc.)
	C. BUILDROOT 		(LINUX KERNEL + ROOTFS (Our python c compiler nano etc apps are in rootfs) )


2.A How to compile and create at91sambootstrap files:
 
   I put at91bootstrap-3-3.8.tar.gz, it is the latest version and works
   unzip to Ubuntu and use it. For any case download link is below:
	
	http://repository.timesys.com/buildsources/a/at91bootstrap-3/

   Go to at91bootstrap-3-3.8 folder and change 

 	/at91bootstrap-3-3.8/board/at91sam9n12ek/at91sam9n12ek.c and .h files
 
   with

 	/at91bootstrap Files/at91sam9n12ek.c and .h
 
   Go to the at91bootstrap-3-3.8 folder location over terminal and apply commands

	1. make mrproper

	2. make at91sam9n12eknf_uboot_defconfig (nf means it will be loaded to nandflash, and u-boot means it will run u-boot)

	3. make menuconfig 			(this will open a configuration window)
	   3.1.Go to RAM Configuration and change RAM size to 64M
	   3.2 Go to NAND Flash configuration and change PMECC Error Correction Bits to 4bits, Sector to 512
	   3.3 Go to U-Boot Image Storage Setup and change External Ram Address to Load U-Boot Image from
               0x26F00000 to 0x22000000
	   3.4 Save an Alternate File and it will create .config file in at91bootstrap-3-3.8.(It is invisible nano .config opens if needed.)

	4. make CROSS_COMPILE=arm-linux-gnueabi-

	RESULT: The at91sam9n12ek-nandflashboot-uboot-3.8-beta1.bin file is ready in binaries file



 2.B How to compile and create u-boot files:

   I put u-boot-2015.1.tar.gz to folder no need to download newer versions since include files naming are changed
   and they are not compiling without error. Unzip it to Ubuntu and use. For any case download link is below:

	 http://repository.timesys.com/buildsources/u/u-boot/

   Go to folder and change

	u-boot-2015.01/board/atmel/at91sam9n12ek/at91sam9n12ek.c file 
	with 
	/u-boot Files/at91sam9n12ek.c

    and

	u-boot-2015.01/include/configs/at91sam9n12ek.h file 
	with	
	/u-boot Files/at91sam9n12ek.h
	
    also For Ubuntu 16.04 gcc5.h, For Ubuntu 18.04 gcc7.h add (they have same code inside as well)

	/u-boot Files/compiler-gcc5.h or gcc.7 file
	to
	u-boot-2015.01/include/linux folder
	

   Go to u-boot-2015.01 folder with terminal and apply commands

	1. make mrproper

	2. make at91sam9n12ek_nandflash_defconfig (nandflash means this file will be written to nandflash)

	3. make CROSS_COMPILE=arm-linux-gnueabi-

	RESULT: The u-boot.bin is in u-boot-2015.01 folder



 2.C How to compile and create build root files:
   
   I put buildroot-2015.2.tar.gz to folder and this version works. For any case download link is below:

	https://buildroot.org/downloads/

   Before start: Buildroot might create errors during compile. 
                 These are mostly package version collision type errors.
                 I look for error file and try to correct them. 
		 If i cannot then i disable that package with "make menuconfig" window.
                 Update when Ubuntu 16.04 is used: In buildroot-2015.2 it gave error i added correction files
		 Update when Ubuntu 18.04 is used: In buildroot-2015.2 it gave error i added correction files


   I used ready linux binary tree files with config files. (Read Linux Embedded Notes 5. DEVICE TREE: part for details of device tree .dts file)
  
   Go to /Buildroot_Files folder and copy 3 files from

	/Buildroot Files/
	to
	/buildroot-2015.02 main folder

   and then rename config-buildroot file as .config (it will be invisible after name change use terminal to see)

   Go to the buildroot-2015.02 folder and apply commands

	(To configure linux kernel to enable drivers use "make linux-menuconfig" command)
   	(for example wifi usb dongle drivers are enabled in linux menuconfig)

	1. make mrproper or make clean

	2. make menuconfig(if i want to add or change something)

	3. make

	RESULT: The files are in buildroot-2015.02/output/images

   Correction if Ubuntu 2016.04 is used (These can be applied when they occur during compilation): 
   	
	if an error not listed below happens first try to remove the installed package from "/output/build" and make again	

	a. ncrses:		copy the code in Buildroot_Correction/ncrse5.9_correction/MKlib_gen.sh 
                                and paste inside of (do not copy paste files itself copy paste code)
                                buildroot-2015.02/output/build/host-crse5.9/ncrse/base/Mklib_gen.sh
	
	c. host-mtd1.5.1: 	(DO THIS TO HOST-MTD NOT MTD folder) copy the code Buildroot_Correction/mkfsutil_correction
                                and paste to (do not copy paste file)
				buildroot-2015.02/output/build/host-mtd1.5.1/mkfs.ubifs/hashtable

   Correction if Ubuntu 2018.04 is with gcc 5 used (These can be applied when they occur during compilation):
   (All error corrections are added below. If could not make it work use Ubuntu 16.04) 
   	
	a. libffi3.1: 		change Buildroot_Correction/libffi3.1_correction/automake file 
				with 
				buildroot-2015.02/output/host/user/bin/automake
	
	b. libglib2.42:         (DO THIS TO LIBGLIB NOT HOST-LIBGLIB folder) change Buildroot_Correction/libglib2.42_correction/gdate.c file
				with
				buildroot-2015.02/output/build/libglib2.42/glib/gdate.c


	c. lzop1.03:            copy the code Buildroot_Correction/lzop1.03_correction/miniacc.f file
                                and paste to (do not copy paste file)
				buildroot-2015.02/output/build/host-lzop1.03/src/miniacc.h

	d. ncrses:		copy the code Buildroot_Correction/ncrse5.9_correction/MKlib_gen.sh 
                                and paste to (do not copy paste file)
                                buildroot-2015.02/output/build/host-crse5.9/ncrse/base/Mklib_gen.sh
	
	e. host-mtd1.5.1: 	(DO THIS TO HOST-MTD NOT MTD folder) copy the code Buildroot_Correction/mkfsutil_correction
                                and paste to (do not copy paste file)
				buildroot-2015.02/output/build/host-mtd1.5.1/mkfs.ubifs/hashtable


	
 3. How to flash files to board:

   Memory locations of files
	
	* at91sam9n12ek-nandflashboot-uboot-3.8.bin -----> 0x00

	* u-boot.bin				     -----> 0x40000

	* at91sam9n12_sam_board_3.15.dtb	     -----> 0x180000

	* uImage.bin				     -----> 0x200000

	* rootfs.ubi				     -----> 0x800000


   Unzip /sam-ba to ubuntu and connect board over usb (see 1) then click connect.
   Select "NandFlash" tab and from Scripts select "Enable NandFlash" click execute button.
   It should print as below:

	-I- NANDFLASH::Init (trace level : 4)
	-I- Loading applet applet-nandflash-sam9n12.bin at address 0x20000000
	-I- Memory Size : 0x20000000 bytes
	-I- Buffer address : 0x20011404
	-I- Buffer size: 0x20000 bytes
	-I- Applet initialization done

   Then again from Scripts select "Pmecc configuration" click execute button.
   (I think it reads nandflash auto but do anyway because when wrong parameters selected it gives error.)
   Select ECC bit: 4
   Select Sector size: 512

   It should print as below:
	-I- Ecc type is 2 Ecc Status is 2
	-I- Configure trimffs 0
	-I- PMECC c0082805 to be Configured
	-I- Pmecc header configration successful
	-I- PMECC configure c0082805   


   3.1. Set Address to 0x00 and from Scripts select "Send Boot File" click execute button.  
      Select at91sam9n12ek-nandflashboot-uboot-3.8.bin file and click open.

      It should print as below:
	   -I- Sending boot file done.

   From now on execute button will not be used. Rest of the files will send with
   "Send File Name" folder selector and address setting.

   3.2. Set Address to 0x40000 and from "Send File Name" folder selector 
      select "u-boot.bin" file and click "Send File".

      It should print as below:
	   -I- Send File /home/ck/Desktop/u-boot.bin at address 0x40000
	   GENERIC::SendFile /home/ck/Desktop/u-boot.bin at address 0x40000
	   -I- File size : 0x5CD30 byte(s)
	   -I- 	Writing: 0x20000 bytes at 0x40000 (buffer addr : 0x20011404)
	   -I- 	0x20000 bytes written by applet
	   -I- 	Writing: 0x20000 bytes at 0x60000 (buffer addr : 0x20011404)
	   -I- 	0x20000 bytes written by applet
	   -I- 	Writing: 0x1CD30 bytes at 0x80000 (buffer addr : 0x20011404)
	   -I- 	0x1CD30 bytes written by applet


   3.3. Set Address to 0x180000 and from "Send File Name" folder selector 
      select "at91sam9n12_sam_board_3.15.dtb" file and click "Send File".
      
      It should print as below:
	   -I- Send File /home/ck/Desktop/at91sam9n12_sam_board_3.15.dtb at address 0x180000
	   GENERIC::SendFile /home/ck/Desktop/at91sam9n12_sam_board_3.15.dtb at address 0x180000
	   -I- File size : 0x2966 byte(s)
	   -I- 	Writing: 0x2966 bytes at 0x180000 (buffer addr : 0x20011404)
	   -I- 	0x20000 bytes written by applet


   3.4. Set Address to 0x200000 and from "Send File Name" folder selector 
      select "uImage.bin" file and click "Send File".
      
      It should print as below:

	   -I- Send File /home/ck/Desktop/uImage.bin at address 0x200000
	   GENERIC::SendFile /home/ck/Desktop/uImage.bin at address 0x200000
	   -I- File size : 0x2B91F0 byte(s)
	   -I- 	Writing: 0x20000 bytes at 0x200000 (buffer addr : 0x20011404)
	   -I- 	0x20000 bytes written by applet
	   ... goes like this
	   -I- 	Writing: 0x191F0 bytes at 0x4A0000 (buffer addr : 0x20011404)
	   -I- 	0x191F0 bytes written by applet


   3.5. Set Address to 0x800000 and from "Send File Name" folder selector 
      select "rootfs.ubi" file and click "Send File".










    
 
