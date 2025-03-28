step 1 : Download the Linux kernel file from the link "https://www.mediafire.com/file/qal35rulrer2k8u/Linux_USB_image.zip/file"
step 2 : Copy the downloaded file from our local PC and paste it in the Host device via FILEZILLA
step 3 : To unzip the file use cmd --> unzip Linux.zip
step 4 : Go "/Linux/code" inside the "Code" folder you will find list of .tar.bz files
step 5 : To unzip and extract .tar.bz2 file use cmd --> tar -xf FILE_NAME
step 6 : Go to "/Linux/tools" inside the "tools" folder you will find list of .tar.bz files extract that using the above cmd

------MAKING UIMAGE AND DTB------
step 1 : Set the enviroinmentel variable using the cmd --> export VARIABLE_NAME=THE_PATH
			(we are using ARM architecture processor so we setting the ARCH as ARM) cmd --> export ARCH=arm
			(locate the gcc compiler library  and set that path as a enviroinmentel vaiable it will locate in "Linux/tools/arm-2011.09/bin/") cmd  --> export CROSS_COMPILE=/home/sanjay/Project/at91sam9x/Linux/tools/arm-2011.09/bin/arm-none-linux-gnueabi- 
step 2 : Go to the Linux root dir ("Linux/code/linux-at91") and run the below commands
step 3 : cmd --> make at91_dt_defconfig
step 4 : cmd --> make menuconfig
step 5 : Before giving "make" cmd need to fix some errors

		Source code Fixes:(go to "Linux/code/linux-at91")
		
			cmd --> cp scripts/dtc/dtc-lexer.lex.c scripts/dtc/dtc-lexer.lex.c.org
			cmd --> vi scripts/dtc/dtc-lexer.lex.c

				 568 // CMS fix - Multiple definition - https://github.com/torvalds/linux/commit/e33a814e772cdc36436c8c188d8c42d019fda639
				 569 // YYLTYPE yylloc;
				 570 extern YYLTYPE yylloc;
				 
				 569 => (-)
				 570 => (+)	
				 
	    Source code Fixes:(go to "Linux/code/linux-at91")	 
		
			cmd --> cp kernel/timeconst.pl kernel/timeconst.pl.org
			cmd --> vi kernel/timeconst.pl

				373   #CMS Fix - Can't use 'defined(@array)' - https://lkml.org/lkml/2012/11/18/159
				374   #if (!defined(@val)) {
				375   if (!@val) {
				376   	@val = compute_values($hz);
				377   }
				
				374 => (-)
				375 => (+)			
step 6 : cmd --> make 
step 7 : cmd --> make dtbs
step 8 : cmd --> cd arch/arm/boot 
step 9 : cmd --> mkimage -A arm -O linux -C none -T kernel -a 20008000 -e 20008000 -n linux-3.6.9 -d zImage uImage 

NOTE : You will find "uImage" and dtb File (for arm AT91SAM9X we use "at91sam9x25ek.dtb" file) in the "arch/arm/boot/" dir. copy that files to your device via Filezilla

------MAKING DEFAULT FILE SYSTEM(UBIFS)------
step 1 : Go to linux root dir
step 2 : cmd --> cd tools
step 3 : To enable executable permission for mkfsimg file run cmd --> chmod +x mkubifsimage
step 4 : cmd --> cp mkubifsimage  ..
step 5 : cmd --> cd ../
step 5 : cmd --> ./mkubifsimage rootfs/ ubifs.img
NOTE : In the root folder the "ubifs.img" will be created in root dir cpy that to your device via Filezilla


------MAKING DEFAULT FILE SYSTEM(JFFS2)------
step 1 : Go to linux root dir
step 2 : cmd -->  mkfs.jffs2 --root=rootfs --output=at-rootfs.jffs2
NOTE : In the root folder the "at-rootfs.jffs2" will be created in root dir cpy that to your device via Filezilla 


********************FLASHING********************

step 1 : Download TFTP using the link "https://bitbucket.org/phjounin/tftpd64/downloads/Tftpd64_SE-4.64-setup.exe" in your local device where the uimage and fs files are located
step 2 : select the tftp sever mode
step 3 : focus the current dir to the dir where we have the Uimage , and kernel related files


SETTING_ENV IN UNIT
-----------------
NOTE: in unit the TFTP is only accessible from the port ETH0 (firtst port E1) the LAN cable should be connected to that port

Set the server ip in uboot , follow the commands
uboot>setenv ipaddr 192.168.10.xxx (your range dont disturb other ips)
uboot>setenv serverip 192.168.10.xxx  (PC Ip)
*If FileSystem JFFS2
uboot>setenv bootargs console=ttyS0,115200 earlyprintk mtdparts=atmel_nand:256k(bootstrap)ro,512k(uboot)ro,256k(env),256k(env_redundant),256k(spare),512k(dtb),6M(kernel)ro,-(rootfs) rootfstype=jffs2 root=/dev/mtdblock7 rw coherent_pool=1M video=800x480-16@60
*If FileSystem UBIFS
uboot>setenv bootargs console=ttyS0,115200 earlyprintk mtdparts=atmel_nand:256k(bootstrap)ro,512k(uboot)ro,256k(env),256k(env_redundant),256k(spare),512k(dtb),6M(kernel)ro,-(rootfs) rootfstype=ubifs ubi.mtd=7 root=ubi0:rootfs rw coherent_pool=1M video=800x480-16@60
uboot>saveenv

LOAD LINUX KERNEL
-----------------
nand erase 0x200000 0x600000

tftp 0x22000000 uImage

nand write 0x22000000 0x200000 <size>

Note : instead of <size> you should put the hex size (0xHEX_VALUE) value got from the tftp cmd you ran


LOAD LINUX FILESYSTEM
---------------------
nand erase 0x800000 0xF800000

*If FileSystem UBIFS
tftp 0x20000000 ubifs.img

*If FileSystem JFFS2
tftp 0x20000000 at-rootfs.jffs2

nand write.trimffs 0x20000000 0x800000 <size>

Note : instead of <size> you should put the hex size (0xHEX_VALUE) value got from the tftp cmd you run


LOAD DTB 
--------
nand erase 0x180000 0x80000

*If unit is AT91SAM9G
tftp 0x21000000 at91sam9g25ek.dtb

*If unit is AT91SAM9X
tftp 0x21000000 at91sam9x25ek.dtb

nand write 0x21000000 0x180000 <size>

Note : instead of <size> you should put the hex size (0xHEX_VALUE) value got from the tftp cmd you run

RESET THE UNIT
-------------
reset

FTP Server Setup in unit(using cmd) (optional) (to download the VSFTPD file in the unit for esblishing FTP connection)
----------------
1) 	In Pc , Open tftp server and focus the folder(RTU-1000) which has FW including vsftp etc
2)	Open console
3) 	ifconfig eth0 192.168.10.(your range) up
4)  tftp -g 192.168.10.xxx(system IP) -l vsftpd -r vsftpd
5)  tftp -g 192.168.10.xxx(system IP) -l vsftpd.conf -r vsftpd.conf
6)	mv vsftpd /bin
7)	chmod +x /bin/vsftpd
8)	mv vsftpd.conf /etc
9)  vsftpd

Setting IP Address
-------------------
cd /etc/network
vi interfaces
Change to the required IP address, gateway and netmask as
address 192.168.10.67
netmask 255.255.255.0
gateway 192.168.10.201

Save the file
!w

Set password
-------------
passwd 
softel
softel

Check for proper Kernel Loaded
------------------------------
uname -a

result --> "Linux corewind 3.6.9 #11 Mon Dec 7 12:57:02 IST 2020 armv5tejl GNU/Linux"

HW Test
-------
10)	Open filezilla and send hwtest to /root in board
11) chmod +x hwtest
12) ./hwTest   -  and follow the steps..

Firmware Loading
----------------
13) Open filezilla and send newInstall.sh & rtuFirmware.tar lib.tar, bldStrswan.tar /root in board
14) In Console 
	cd /root
	chmod +x newInstall.sh 
	./newInstall.sh

15) The unit will reboot upon successful installation of firmware		  



EXTRA CMDS :
tftp -g 192.168.10.120 -l serPortRecvHighSpeed -r serPortRecvHighSpeed
tftp -g 192.168.10.120 -l ser_send -r ser_send
tftp -g 192.168.10.120 -l serPortRecv -r serPortRecv

to link some dot .files --> ln -s <DOT_FILE_NAME> <LINKING_FILE_NAME>
convert dtb_to_dts --> dtc -I dtb -O dts <DTB_FILE_NAME> -o  <DTS_FILE_NAME>
convert dts_to_dtb --> dtc -I dts -O dtb -f <DTS_FILE_NAME> -o <DTB_FILE_NAME>



IMPORTANT NOTES FOR DRIVERS : 
	1 - In menuconfig it is mandatory to enable Watch dog if it's not defaultly enabled (loc ---> device drivers ->watch dog time support )
	2 - for using RTC enable [Dallas/Maxim DS1307/37/38/39/40, ST M41T00, EPSON RX-8025] (loc ---> device drivers -> Real time clock -> Dallas/Maxim DS1307/37/38/39/40, ST M41T00, EPSON RX-8025)
	3 - after making dtb files convert the dtb file (loc --> arch/arm/boot) to dts file using cmd --> dtb_to_dts --> dtc -I dtb -O dts <DTB_FILE_NAME> -o  <DTS_FILE_NAME>
		
		comment these lines only if its available
		
			70		// m25p80@0 {
			71		//	compatible = "atmel,at25df321a";
			72		//	spi-max-frequency = <0x2faf080>;
			73		//	reg = <0x00>;
			74		// };
			
			
		check watch dog is enabled or not . The status key should be "okay". (IF IT IS NOT DONE IT WILL CAUSE PROBLEM IN SERIAL COMMUNIACTION)
		
			821  	watchdog@fffffe40 {	
			822  	compatible = "atmel,at91sam9260-wdt";	
			823  	reg = <0xfffffe40 0x10>;	
			824  	status = "okay";
			825  	};
		
		
		To enable external RTC the line numer(170 - 173) this in specified line below (inside i2c@0 structure)
		
			160   i2c@0 {	
			161		compatible = "i2c-gpio";	
			162		gpios = <0x04 0x1e 0x00 0x04 0x1f 0x00>;	
		    163		i2c-gpio,sda-open-drain;	
			164		i2c-gpio,scl-open-drain;	
			165		i2c-gpio,delay-us = <0x05>;	
			166		#address-cells = <0x01>;	
			167		#size-cells = <0x00>;	
			168		status = "okay";
			169
			170		rtc@68 {	
			171		compatible = "dallas/maxim,ds1307";	
			172		reg = <0x68>;	
			173		};	
			174	  };
		
		Finally convert DTS to DTB file using cmd --> dtc -I dts -O dtb -f <DTS_FILE_NAME> -o <DTB_FILE_NAME>

	4 - FOR FIBOCOMM 4G MODEM in AT91SAM9X
		In menuconfig enable these options
			Device Drivers -> Network device support menu -> PPP(point to point protocol) support
			Device Drivers -> USB support -> USB Serial Converter support -> USB driver for GSM and CDMA modems
			Device Drivers -> Network device support -> USB NetworkAdapters -> Multi-purpose USB Networking Framework
			Device Drivers -> Network device support -> USB NetworkAdapters -> CDC Ethernet support (smart devices such as cable modems)
		
		Drive source code modification
		
			OPEN "drivers/usb/serial/option.c" FILE in edit mode
			
				Add the below code at the macro area in the file.
				
					#define FIBOCOM_VENDOR_ID 0x2cb7
					#define FIBOCOM_PRODUCT_L71X 0x0001
					#define FIBOCOM_USB_VENDOR_AND_INTERFACE_INFO(vend, cl, sc, pr) \
					.match_flags = USB_DEVICE_ID_MATCH_INT_INFO \
					| USB_DEVICE_ID_MATCH_VENDOR, \
					.idVendor = (vend), \
					.bInterfaceClass = (cl), \
					.bInterfaceSubClass = (sc), \
					.bInterfaceProtocol = (pr)
				
				Add the pid information to be configured into the static structure array - static const struct 
				usb_device_id option_ids[]. Find the last line containing "ZTE_VENDOR_ID" in this array, and add the 
				following below codes
				
					{ FIBOCOM_USB_VENDOR_AND_INTERFACE_INFO(FIBOCOM_VENDOR_ID, 0xff, 0xff, 0xff) },
					{ FIBOCOM_USB_VENDOR_AND_INTERFACE_INFO(FIBOCOM_VENDOR_ID, 0x0a, 0x00, 0xff) },
					{ USB_DEVICE_AND_INTERFACE_INFO(0x19d2, 0x0256, 0xff, 0xff, 0xff) },
					{ USB_DEVICE_AND_INTERFACE_INFO(0x19d2, 0x0579, 0xff, 0xff, 0xff) },
			
			OPEN "drivers/usb/serial/usb_wwan.c" File in edit mode
			
				Add the below macro code:
				
					#define FIBOCOM_BCDUSB 0x0100
					#define FIBOCOM_VENDOR_ID 0x2cb7
					
							loc to add lines -->
							
							#include <linux/serial.h>
							#include "usb-wwan.h"

							// ********************** SANJAY ADDED BEGIN *********************
							#define FIBOCOM_BCDUSB 0x0100
							#define FIBOCOM_VENDOR_ID 0x2cb7
							// ********************** SANJAY ADDED END **********************

							static bool debug;

							void usb_wwan_dtr_rts(struct usb_serial_port *port, int on)

				Add the below lines in the "usb_wwan_write" function:
				
					struct usb_host_endpoint *ep;
					 
					if((FIBOCOM_VENDOR_ID == port->serial->dev->descriptor.idVendor)
					&& (FIBOCOM_BCDUSB != port->serial->dev->descriptor.bcdUSB)) { ep = 
					usb_pipe_endpoint(this_urb->dev, this_urb->pipe);
					if (ep && (0 != this_urb->transfer_buffer_length)
					&& (0 == this_urb->transfer_buffer_length
					% ep->desc.wMaxPacketSize)) {
					this_urb->transfer_flags |= URB_ZERO_PACKET;
					printk("GHT: Send ZERO PACKET ####\r\n");
					}
					}
					
							loc to add lines -->
							
							int usb_wwan_write(struct tty_struct *tty, struct usb_serial_port *port, const unsigned char *buf, int count)
							{
									struct usb_wwan_port_private *portdata;
									struct usb_wwan_intf_private *intfdata;
									int i;
									int left, todo;
									struct urb *this_urb = NULL; /* spurious */
									int err;
									unsigned long flags;

									//************* SANJAY ADDED BEGIN ***************
									struct usb_host_endpoint *ep;
									//************ SANJAY ADDED END ****************

									portdata = usb_get_serial_port_data(port);
									intfdata = port->serial->private;

									dbg("%s: write (%d chars)", __func__, count);

									..............
									..............
									..............
									
									if (err < 0)
									break;

									/* send the data */
									memcpy(this_urb->transfer_buffer, buf, todo);
									this_urb->transfer_buffer_length = todo;

									//************* SANJAY ADDED BEGIN ***************
									if ((FIBOCOM_VENDOR_ID == port->serial->dev->descriptor.idVendor) && (FIBOCOM_BCDUSB != port->serial->dev->descriptor.bcdUSB))
									{
											ep =
													usb_pipe_endpoint(this_urb->dev, this_urb->pipe);
											if (ep && (0 != this_urb->transfer_buffer_length) && (0 == this_urb->transfer_buffer_length % ep->desc.wMaxPacketSize))
											{
													this_urb->transfer_flags |= URB_ZERO_PACKET;
													printk("GHT: Send ZERO PACKET ####\r\n");
											}
									}
									//************** SANJAY ADDED END ****************

									spin_lock_irqsave(&intfdata->susp_lock, flags);
									if (intfdata->suspended)
									{
											usb_anchor_urb(this_urb, &portdata->delayed);
											spin_unlock_irqrestore(&intfdata->susp_lock, flags);
									}
	
EXTRA FEATURES :

	FTP Server Setup
	----------------
		1) 	In Pc , Open tftp server and focus the folder(RTU-1000) which has FW including vsftp etc
		2)	Open unit console
		3) 	cmd --> ifconfig eth0 192.168.10.xxx up 
			(xxx - set your ip range dont disturb other ips) the above cmd is used to set the ip address for ethernet port 0
		4)  cmd --> tftp -g 192.168.10.xxx(server IP) -l vsftpd -r vsftpd
		5)  cmd --> tftp -g 192.168.10.xxx -l vsftpd.conf -r vsftpd.conf
		6)	cmd --> mv vsftpd /bin
		7)	cmd --> chmod +x /bin/vsftpd
		8)	cmd --> mv vsftpd.conf /etc
		9)  cmd --> vsftpd
			Now the FTP server in unit is ready you can access the server from the FILEZILLA 
			NOTE: it is mandatory to set password for ftp only by that it can able to access by FILEZILLA