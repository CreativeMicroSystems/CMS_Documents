##################################################
############ UPDATED on 17-APR-2025 ##############
##################################################

SETTING_ENV IN SERVER:
---------------
    cmd --> export ARCH=arm
    cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
    cmd --> export imxcc=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
	
Compile Linux and DTB:
----------------------
    cmd --> cd MYiR-iMX-Linux/
    cmd --> make myd_y6ulx_defconfig
	cmd --> make menuconfig
	cmd --> make
	
	need to fix some errors (NOTE : the dtc-lexer.lex.c.org will be created only after giving make cmd)
	
	Source code Fixes:(go to "Linux/code/linux-at91")
		
			cmd --> cp scripts/dtc/dtc-lexer.lex.c scripts/dtc/dtc-lexer.lex.c.org
			cmd --> vi scripts/dtc/dtc-lexer.lex.c

				 568 // CMS fix - Multiple definition - https://github.com/torvalds/linux/commit/e33a814e772cdc36436c8c188d8c42d019fda639
				 569 // YYLTYPE yylloc;
				 570 extern YYLTYPE yylloc;
				 
				 569 => (-)
				 570 => (+)	
	
	KERNEL IMAGE:
		In IMX the kernel image is ZIMAGE file
		zImage location --> /IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/04-Source/MYiR-iMX-Linux/arch/arm/boot
	
	DTB:
		myd-y6ull-gpmi-weim.dtb (FOR Y2 BASED BOARDS)
	    myd-y6ul-gpmi-weim.dtb  (FOR G2 BASED BOARDS)
		dtb file location --> /IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/04-Source/MYiR-iMX-Linux/arch/arm/boot/dts


***************************** FLASHING USING TFTP BEGIN ******************************

step 1 : Download TFTP using the link "https://bitbucket.org/phjounin/tftpd64/downloads/Tftpd64_SE-4.64-setup.exe" in your local device where the uimage and fs files are located
step 2 : Select the tftp sever mode
step 3 : Focus the current dir to the dir where we have the Uimage , and kernel related files

SETTING_ENV IN UNIT
-----------------
NOTE: in unit the TFTP is only accessible from the port 1 (first port E1) the LAN cable should be connected to that port

cmd --> setenv ipaddr 192.168.10.xxx (your range dont disturb other ips)
cmd --> setenv serverip 192.168.10.xxx  (PC Ip)
cmd --> setenv ethaddr 32:b0:29:c4:e3:ff (any MAC address)
cmd --> saveenv
cmd --> reset

cmd --> pri
Execute "pri" command in uboot and check for bootcmd variable ("bootcmd=nand read ${loadaddr} 0x500000 0xA00000;nand read ${fdt_addr} 0xF00000 0x100000;bootz ${loadaddr} - ${fdt_addr}")
If the varibale contains the address 0x500000 0xA00000. Then follow the Method 1 for flashing the kernel manually.
If the varibale contains the address 0x600000 0xA00000. Then follow the Method 2 for flashing the kernel manually.
	
	
Method 1:
---------
Start Method 1---------------------------------------   

LOAD LINUX KERNEL:
	cmd --> tftp ${loadaddr} zImage
	cmd --> nand erase 0x500000 0xA00000
	cmd --> nand write ${loadaddr} 0x500000 0xA00000

LOAD DTB:
	(FOR Y2 BASED BOARDS) cmd --> tftp ${fdt_addr} myd-y6ull-gpmi-weim.dtb
					(OR)
	(FOR G2 BASED BOARDS) cmd --> tftp ${fdt_addr} myd-y6ul-gpmi-weim.dtb
	
	cmd --> nand erase 0xF00000 0x100000
	cmd --> nand write ${fdt_addr} 0xF00000 0x100000

LOAD LINUX FILESYSTEM: 	
	cmd --> tftp ${initrd_addr} ubi.img
	cmd --> nand erase 0x1000000 0xF000000
	cmd --> nand write.e ${initrd_addr}  0x1000000 <size e.g 0x4340000> 

End Method 1---------------------------------------   

			 
Method 2:
---------
Start Method 2---------------------------------------   

LOAD LINUX KERNEL:
	cmd --> tftp ${loadaddr} zImage
	cmd --> nand erase 0x600000 0xA00000
	cmd --> nand write ${loadaddr} 0x600000 0xA00000

LOAD DTB:
	(FOR Y2 BASED BOARDS) cmd --> tftp ${fdt_addr} myd-y6ull-gpmi-weim.dtb
				(OR)
	(FOR G2 BASED BOARDS) cmd --> tftp ${fdt_addr} myd-y6ul-gpmi-weim.dtb
	
	cmd --> nand erase 0x1000000 0x100000
	cmd --> nand write ${fdt_addr} 0x1000000 0x100000

LOAD LINUX FILESYSTEM: 
	cmd --> tftp ${initrd_addr} ubi.img
	cmd --> nand erase 0x1100000 0xEF00000
	cmd --> nand write.e ${initrd_addr} 0x1100000 <size e.g 0x4340000>

End Method 2---------------------------------------   
	
***************************** FLASHING USING TFTP END ******************************
	
	
***************************** CROSS COMPILATION BEGIN *******************************
	
	--------- OPEN SSL ----------
		Download openssl.tar file using link "https://www.openssl.org/source/openssl-1.1.1w.tar.gz"
		Transfer to the server using FTP
		cmd --> cd openssl-1.1.1w
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd -->	./Configure linux-armv4 --prefix=$(pwd)/openssl/
				openssl is nothing but the folder name to create to store the lib and include files
		cmd --> make CC=$imxcc
		cmd --> make install
		
		
	--------- PAHO MQTT ----------
		cmd --> git clone https://github.com/eclipse/paho.mqtt.c.git 
		cmd --> cd paho.mqtt.c
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> export LDFLAGS='-L<OPENSSL LIBRARY PATH>'   
				e.g. -> export LDFLAGS='-L/home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/lib'
		cmd --> export CPPFLAGS='-I<OPENSSL INCLUDE PATH>'   
				e.g. -> export CPPFLAGS='-I/home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/include'
		cmd --> vi Makefile
			
			161 # CCFLAGS_SO = -g -fPIC $(CFLAGS) -D_GNU_SOURCE -Os -Wall -fvisibility=hidden -I$(blddir_work) -DPAHO_MQTT_EXPORTS=1
			162 CCFLAGS_SO = -g -fPIC $(CFLAGS) -D_GNU_SOURCE -Os -Wall -fvisibility=hidden -I$(blddir_work) -I /home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/include -DPAHO_MQTT_EXPORTS=1

			201 #EXTRA_LIB = -ldl
			202 EXTRA_LIB = -ldl -L/home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/lib

			205 #CCFLAGS_SO += -Wno-deprecated-declarations -DOSX -I /usr/local/opt/openssl/include
			206 CCFLAGS_SO += -Wno-deprecated-declarations -DOSX -I /home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/include

			208 LDFLAGS_C += -Wl,-install_name,lib$(MQTTLIB_C).so.${MAJOR_VERSION}

			210 #LDFLAGS_CS += -Wl,-install_name,lib$(MQTTLIB_CS).so.${MAJOR_VERSION} -L /usr/local/opt/openssl/lib
			211 LDFLAGS_CS += -Wl,-install_name,lib$(MQTTLIB_CS).so.${MAJOR_VERSION} -L /home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/lib

			213 LDFLAGS_A += -Wl,-install_name,lib${MQTTLIB_A}.so.${MAJOR_VERSION}

			215 #LDFLAGS_AS += -Wl,-install_name,lib${MQTTLIB_AS}.so.${MAJOR_VERSION} -L /usr/local/opt/openssl/lib
			216 LDFLAGS_AS += -Wl,-install_name,lib${MQTTLIB_AS}.so.${MAJOR_VERSION} -L /home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/lib

			161 => (-)  162 => (+)
			201 => (-)  202 => (+)
			205 => (-)  206 => (+)
			210 => (-)  211 => (+)
			215 => (-)  216 => (+)
			
		cmd --> make CC=$imxcc
	
	--------------- ZLIB -------------
		cmd --> download .rpm file using link http://rpmfind.net/linux/KDE/unstable/server/kolab/kolab-server-1.0/src/zlib-1.1.4-20030227.src.rpm
		cmd --> extract that and get .tar.gz file , tranfer that file through file zilla
		cmd --> tar -xvf zlib-1.1.4.tar.gz
		cmd --> cd zlib-1.1.4
		cmd --> ./configure --prefix=$(pwd)/zlib
				In the above cmd the prefix is the path name where to store the cross compiled files also here we should not give HOST=arch name 
		cmd --> mkdir zlib
		cmd --> make CC=$imxcc 
		cmd --> make install
		
	-------------- 	CURL WITHOUT SSL AND ZLIB-----------
		cmd --> git clone https://github.com/curl/curl.git
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> ./buildconf
		cmd --> ./configure CC=$imxcc --prefix=$(pwd)/curl  --host=$ARCH  --without-ssl
				In the above cmd the prefix is the path name where to store the cross compiled files
		cmd --> make
		cmd --> make install
	
	
	-------------- 	CURL WITH SSL AND WITH ZLIB-----------
		cmd --> git clone https://github.com/curl/curl.git
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> export LDFLAGS='-L<OPENSSL LIBRARY PATH> -L<ZLIB LIBRARY PATH>'   
				e.g. -> export LDFLAGS='-L/home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/lib -L/home/sanjay/cross_compilation/zlib/zlib-1.1.4/zlib/lib'
		cmd --> export CPPFLAGS='-I<OPENSSL INCLUDE PATH> -I<ZLIB INCLUDE PATH>'   
				e.g. -> export CPPFLAGS='-I/home/sanjay/cross_compilation/openssl/openssl-1.1.1w/openssl/include -I/home/sanjay/cross_compilation/zlib/zlib-1.1.4/zlib/include'
		cmd --> ./buildconf
		cmd --> ./configure CC=$imxcc --prefix=$(pwd)/curl  --host=$ARCH  --with-ssl --with-zlib
				In the above cmd the prefix is the path name where to store the cross compiled files
		cmd --> make
		cmd --> make install 
		Referenc : https://www.matteomattei.com/how-to-cross-compile-curl-library-with-ssl-and-zlib-support/
		
		
	------------- GMP ------------------
		cmd --> wget https://ftp.gnu.org/gnu/gmp/gmp-6.2.0.tar.bz2
		cmd --> tar -xvf gmp-6.2.0.tar.bz2
		cmd --> ./configure CC=$imxcc --host=arm --prefix=$(pwd)/output
		cmd --> make
		cmd --> make install
		
		
	-------------- STRONG SWAN ------------
		cmd --> export LDFLAGS='-L/home/sanjay/cross_compilation/gmp/gmp-6.2.0/output/lib'
		cmd --> export CPPFLAGS='-I/home/sanjay/cross_compilation/gmp/gmp-6.2.0/output/include'
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> wget https://download.strongswan.org/strongswan-5.8.2.tar.bz2
		cmd --> tar -xvf strongswan-5.8.2.tar.bz2
		cmd --> cd strongswan-5.8.2/
		cmd --> ./configure CC=$imxcc --host=$ARCH --prefix=$(pwd)/strongswan --disable-shared --enable-static --enable-monolithic
		cmd --> make
		cmd --> make install
				Note : while giving the above cmd it will give some error but it will create a files in the specified destination path.
		
	------------- VSFTPD -----------------
		Download openssl.tar file using link "https://security.appspot.com/downloads/vsftpd-3.0.5.tar.gz"
		Transfer to the server using FTP
		cmd --> tar -xvf vsftpd-3.0.5.tar.gz
		cmd -->  cd vsftpd-3.0.5/
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> vi Makefile
		
				2 #CC      =       gcc
				3 CC      =       $(imxcc)
				
				11 #LIBS   =       `./vsf_findlibs.sh`
				12 LIBS =-lcap-lpam
				
				48  $(INSTALL) -m 644 xinetd.d/vsftpd /etc/xinetd.d/vsftpd; fi
				49  $ (INSTALL)-d-m 755/home/vsftpd/sbin/;
				50  $ (INSTALL)-M 755 vsftpd/home/vsftpd/sbin/vsftpd;


				2 =>  (-) 3 => (+)
				11 => (-) 12 => (+)
				49 => (+)
				50 => (+)
				
		cmd --> make
		cmd --> make install
	
	------------- DHCP -----------------
		cmd --> export ARCH=arm
		cmd --> export imxcc=/home/sanjay/Project/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> cd /home/sanjay/CC/cross_compilation_imx/dhcp
		cmd --> wget https://src.fedoraproject.org/repo/pkgs/dhcp/dhcp-4.3.0b1.tar.gz/1cde1782c62af89610d9c73035b1e288/dhcp-4.3.0b1.tar.gz
		cmd --> wget http://wiki.beyondlogic.org/patches/dhcp-4.3.0b1.bind_arm-linux-gnueabi.patch
		cmd --> wget http://wiki.beyondlogic.org/patches/bind-9.9.5rc1.gen_crosscompile.patch
		cmd --> tar -xzf dhcp-4.3.0b1.tar.gz
		cmd --> cd dhcp-4.3.0b1
		cmd --> patch -p1 < ../dhcp-4.3.0b1.bind_arm-linux-gnueabi.patch 
		cmd --> cd bind
		cmd --> tar -xzf bind.tar.gz
		cmd --> cd bind-9.9.5rc1
		cmd --> patch -p1 < ../../../bind-9.9.5rc1.gen_crosscompile.patch 
		cmd --> cd ../..
		cmd --> ./configure CC=$imxcc --host=$ARCH --prefix=$(pwd)/dhcp_build_imx --build=armv7l ac_cv_file__dev_random=yes 
		cmd --> make CC=$imxcc
		cmd --> make install
	
	-------------- DNSMASQ ------------
		cmd --> wget http://www.thekelleys.org.uk/dnsmasq/dnsmasq-2.89.tar.gz
		cmd --> tar -xvf dnsmasq-2.89.tar.gz
		cmd --> cd dnsmasq-2.89
		cmd --> export ARCH=arm
		cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
		cmd --> export CC=${CROSS_COMPILE}gcc
		cmd --> export CXX=${CROSS_COMPILE}g++
		cmd --> export AR=${CROSS_COMPILE}ar
		cmd --> export LD=${CROSS_COMPILE}ld
		cmd --> export RANLIB=${CROSS_COMPILE}ranlib
		cmd --> make clean
		cmd --> make CC=${CROSS_COMPILE}gcc
		cmd --> cd src 
				The "dnsmasq" exe file will be there in the src
	
	------------ libnl --------
		cmd --> mkdir libnl
		cmd --> cd libnl
		cmd --> wget https://github.com/thom311/libnl/releases/download/libnl3_7_0/libnl-3.7.0.tar.gz
		cmd --> tar xf libnl-3.7.0.tar.gz
		cmd --> cd libnl-3.7.0
		cmd --> cd ~/CC/cross_compilation_imx/libnl/libnl-3.7.0
		cmd --> export ARCH=arm
		cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
		cmd --> export CC=${CROSS_COMPILE}gcc
		cmd --> export CXX=${CROSS_COMPILE}g++
		cmd --> export AR=${CROSS_COMPILE}ar
		cmd --> export LD=${CROSS_COMPILE}ld
		cmd --> export RANLIB=${CROSS_COMPILE}ranlib
		cmd --> ./configure --host=arm-linux-gnueabihf --prefix=/usr --with-sysroot=$SYSROOT
		cmd --> make -j$(nproc)
		cmd --> make DESTDIR=$(pwd)/libnl_output install
		
	------------ hostapd --------
		cmd --> mkdir hostapd
		cmd --> cd hostapd
		cmd --> wget https://w1.fi/releases/hostapd-2.11.tar.gz
		cmd --> tar xvf hostapd-2.11.tar.gz
		cmd --> cd hostapd-2.11/hostapd
		cmd --> export CFLAGS="-I/home/sanjay/CC/cross_compilation_imx/libnl/libnl-3.7.0/libnl_output/usr/include -I/home/sanjay/CC/cross_compilation_imx/openssl/openssl-1.1.1w/openssl/include"
		cmd --> export LDFLAGS="-L/home/sanjay/CC/cross_compilation_imx/libnl/libnl-3.7.0/libnl_output/usr/lib -L/home/sanjay/CC/cross_compilation_imx/openssl/openssl-1.1.1w/openssl/lib"
		cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
		cmd --> export CC=${CROSS_COMPILE}gcc
		cmd --> export CXX=${CROSS_COMPILE}g++
		cmd --> export AR=${CROSS_COMPILE}ar
		cmd --> export LD=${CROSS_COMPILE}ld
		cmd --> export RANLIB=${CROSS_COMPILE}ranlib
		cmd --> make clean
		cmd --> cp defconfig .config
		cmd --> make CC=${CROSS_COMPILE}gcc DESTDIR=$(pwd)/hostapd_output
		cmd --> mkdir hostapd_output
		cmd --> mv hostapd hostapd_cli hostapd_output/
		cmd --> cd hostapd_output
		cmd --> ls -al
			    Transfer that "hostapd" & "hostapd_cli" to the IMX device's /bin folder
		cmd --> cd /home/sanjay/CC/cross_compilation_imx/openssl/openssl-1.1.1w/openssl/lib
				Tranfer the "libcrypto.so" "libcrypto.so.1.1" "libssl.so.1.1"  to device's /lib folder
				
		Note : After transferring put [CMD -> ldconfig]  in IMX device (to refresh the lib folders)
	
	------------ libmodbus -----------
		########## Compiled Enviroinment ##########
		Distributor ID      : Ubuntu
		Description         : Ubuntu 22.04.3 LTS
		Release             : 22.04
		Codename            : jammy
		Linux Kernel version: 6.5.0-26-generic
		########## Compiled Enviroinment ##########

		cmd --> mkdir libmodbus
		cmd --> cd libmodbus
		cmd --> wget https://github.com/stephane/libmodbus/releases/download/v3.1.10/libmodbus-3.1.10.tar.gz
		cmd --> tar xvf libmodbus-3.1.10.tar.gz
		cmd --> cd libmodbus-3.1.10
		cmd --> export ARCH=arm
		cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
		cmd --> mkdir -p ./output
		cmd --> ./configure --host=arm-linux-gnueabihf --prefix=$(pwd)/output CC=${CC} CFLAGS="-static -mfloat-abi=hard" LDFLAGS="-Wl,--dynamic-linker=/lib/ld-linux-armhf.so.3"
		cmd --> make -j$(nproc)
		cmd --> make install
		transfer the files in the lib to the device "/lib" dir and give the permissions
		cmd --> cd output/lib
		use the .h files in the "./include/modbus" dir as a header for development
		cmd --> cd output/include/modbus/
		Note:
			Default test files will be generated in the "./tests" folder


		--- for compiling Sample files BEGIN ----
		cmd --> export CC=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		cmd --> export LIBMODBUS_LIB_PATH=/home/sanjay/CC/cross_compilation_imx/libmodbus/libmodbus-3.1.10/output/lib
		cmd --> export LIBMODBUS_INC_PATH=/home/sanjay/CC/cross_compilation_imx/libmodbus/libmodbus-3.1.10/output/include/modbus
		cmd --> cat <<EOF > sample_modtcp.c
				#include <stdio.h>
				#include <modbus.h>

				int main(void) {
				  printf("#### sample_modbus started running .... ####\n");
				  modbus_t *mb;
				  uint16_t tab_reg[32];
				  mb = modbus_new_tcp("127.0.0.1", 1502);
				  modbus_connect(mb);

				  /* Read 5 registers from the address 0 */
				  modbus_read_registers(mb, 0, 5, tab_reg);

				  modbus_close(mb);
				  modbus_free(mb);
				  printf("#### sample_modbus Stopped ####\n");
				}
				EOF

		cmd --> $CC sample_modtcp.c -I. -I$LIBMODBUS_INC_PATH -L$LIBMODBUS_LIB_PATH -lmodbus -o sample_modtcp_exe -Wl,--dynamic-linker=/lib/ld-linux-armhf.so.3
		--- for compiling Sample files END ----
		
	------------ python 3.9 [NOT COMPLETED]--------
		cmd --> mkdir python3.9
		cmd --> wget https://www.python.org/ftp/python/3.9.18/Python-3.9.18.tgz
		cmd --> tar -xvzf Python-3.9.18.tgz
		cmd --> cd Python-3.9.18
		cmd --> export CROSS_COMPILE=/home/sanjay/KERNEL/imx6ul/IMX6UL/MYIR-MYD-Y6ULG2/MYD-Docs/03-Tools/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-

		
***************************** CROSS COMPILATION END *******************************


IMPORTANT NOTES FOR ENABLING DRIVERS : 
	1 - In menuconfig it is mandatory to enable Watch dog if it's not defaultly enabled (loc ---> device drivers ->watch dog time support )
	2 - for using RTC enable [Dallas/Maxim DS1307/37/38/39/40, ST M41T00, EPSON RX-8025] (loc ---> device drivers -> Real time clock -> Dallas/Maxim DS1307/37/38/39/40, ST M41T00, EPSON RX-8025)
	3 - for using  RTL8188EUS WIFI Module enable "Realtek RTL8192CU/RTL8188CU USB Wireless Network Adapter" and "Debugging output for rtlwifi driver family" (loc ---> Device Drivers->Network device support->Wireless LAN->Realtek rtlwifi family of devices)
	4 - for using RTL8731BU WIFI Modulde there is no default drivers present in linux4.1.15 so we want to add manuallay
		*********** Manual driver compilation for RTL8731BU *********
		SETTING_ENV IN SERVER:
		---------------
			cmd --> export ARCH=arm
			cmd --> export CROSS_COMPILE=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
			cmd --> export imxcc=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
		
		IMPORTANAT NOTE:
			WE HAVE A DRIVER SOURCE FILE FOR "RTL8733BU" CHIP. ALL RTL873X CAN USE THIS SO NOW WE ARE COMPILING THIS FOR "RTL8731BU" CHIPSET. 
			
		step 1 :  create a directory in your server
					cmd -> mkdir ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES
		step 2 :  get the  "old_DCU_kernel_src.7z" from the https://github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES and put it inside "~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES" dir
		step 3 :  get the  "rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e.tar.gz" from the https://github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES and put it inside "~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES" dir
		step 4 :  extract the files
					cmd -> 7z x old_DCU_kernel_src.7z
					cmd -> tar -xvf rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e.tar.gz 
		step 5 :  Making Wifi driver 
					Enabling wifi supporting driver cfgs(cfg80211) in kernel src code
						cmd --> cd ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux
						cmd --> make menuconfig
						enable drivers
							--- Networking support---
								--- Wireless ---
									<*>   cfg80211 - wireless configuration API                                                                                     
									[ ]     nl80211 testmode command (NEW)                                                                          
									[ ]     enable developer warnings (NEW)                                                                                                         
									[ ]     cfg80211 regulatory debugging (NEW)                                                                                                        
									[ ]     cfg80211 certification onus (NEW)                                                                                                           
									[*]     enable powersave by default (NEW)                                                                                                           
									[ ]     cfg80211 DebugFS entries (NEW)                                                                                                              
									[ ]     use statically compiled regulatory rules database (NEW)                                                                                     
									[ ]     cfg80211 wireless extensions compatibility (NEW)                                                                                            
									<*>   Generic IEEE 802.11 Networking Stack (mac80211)                                                                                               
									[*]   Minstrel (NEW)                                                                                                                                
									[*]     Minstrel 802.11n support (NEW)                                                                                                              
									[ ]       Minstrel 802.11ac support (NEW)                                                                                                          
									Default rate control algorithm (Minstrel)  --->                                                                                              
									[ ]   Enable mac80211 mesh networking (pre-802.11s) support (NEW)                                                                                   
									[ ]   Enable LED triggers (NEW)                                                                                                                     
									[ ]   Export mac80211 internals in DebugFS (NEW)                                                                                                    
									[ ]   Trace all mac80211 debug messages (NEW)                                                                                                       
									[ ]   Select mac80211 debugging features (NEW)  ----
						save the menuconfig
					Make the kernel first then make the wifi driver
						cmd --> make
							If everything goes well, it will produce a kernel files.
							
							KERNEL IMAGE:
								In IMX the kernel image is ZIMAGE file
								zImage location --> ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux/arch/arm/boot
							
							DTB:
								myd-y6ull-gpmi-weim.dtb (FOR Y2 BASED BOARDS)
								myd-y6ul-gpmi-weim.dtb  (FOR G2 BASED BOARDS)
								dtb file location --> ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux/arch/arm/boot/dts
								
					Now make the wifi driver			
					cmd --> cd ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e
					cmd --> vi Makefile
							152 CONFIG_PLATFORM_I386_PC = y  (--)
							152 CONFIG_PLATFORM_I386_PC = n  (++)
							
							167 CONFIG_PLATFORM_FS_MX61 = n  (--)
							167 CONFIG_PLATFORM_FS_MX61 = y  (++)
							
							--- remove the below lines ---
								1629 ifeq ($(CONFIG_PLATFORM_FS_MX61), y)
								1630 EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
								1631 ARCH := arm
								1632 CROSS_COMPILE := /home/share/CusEnv/FreeScale/arm-eabi-4.4.3/bin/arm-eabi-
								1633 KSRC ?= /home/share/CusEnv/FreeScale/FS_kernel_env
								1634 endif
							--- remove the above lines ---
							
							+++ add the below line +++
								1629 ifeq ($(CONFIG_PLATFORM_FS_MX61), y)
								1630 EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN -DRTW_USE_CFG80211_STA_EVENT -DCONFIG_IOCTL_CFG80211
								1631 ARCH := arm
								1632 CROSS_COMPILE := /home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
								1633 KSRC ?= /home/imx6ul/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux
								1634 MODULE_NAME := rtl8733bu_driver
								1635 endif
							+++ add the above line +++
							NOTE:
								KSRC is a kernel source so if you are compiling this driver to some kernel don't forget to give that kernel source path
								or else it will cause kernel mismatch error
								
								In the makefile you can give thew output .ko file name in "MODULE_NAME"
								
					cmd --> make clean
					cmd --> make
							If everything goes well, it will produce a rtl8733bu_driver.ko file.
				
				
EXTRA FEATURES (Establishing in Device): 

	FTP Server Setup (for file transfer)
	----------------
		1) 	In Pc , Open tftp server and focus the folder(RTU-1000) which has FW including vsftp etc
		2)	Open unit console
		3) 	cmd --> ifconfig eth0 192.168.10.xxx up
			(xxx - set your ip range dont disturb other ips) (eth0 (or) eth1 based on which the lan cable is connected)the above cmd is used to set the ip address for ethernet port 0
		4)  cmd --> tftp -g 192.168.10.xxx(server IP) -l vsftpd -r vsftpd
		5)  cmd --> tftp -g 192.168.10.xxx -l vsftpd.conf -r vsftpd.conf
		6)	cmd --> mv vsftpd /bin
		7)	cmd --> chmod +x /bin/vsftpd
		8)	cmd --> mv vsftpd.conf /etc
		9)  cmd --> vsftpd
			Now the FTP server in unit is ready you can access the server from the FILEZILLA by giving the IP address set for eth 0 and pasword
			NOTE: It is mandatory to set password for ftp, only by that it can able to access by FILEZILLA.
			
	RTL8188EUS WIFI Module Setup (for wifi connection)
	----------------
		------------ CHECK WIFI MODULE HARDWARE CONNECTION STATUS VIA USB ------------
		cmd --> lsusb
		eg:  you will get the below lines :
			Bus 002 Device 001: ID 1d6b:0002
			Bus 001 Device 002: ID 0bda:8176 
			Bus 001 Device 001: ID 1d6b:0002
			The Device 002 in the Bus001 is a RTL8188EUS USB port its produxt and vendor id is : 0bda:8176 
		cmd --> ifconfig -a
			eg: you will find the below lines
				wlan0   Link encap:Ethernet  HWaddr 00:1d:43:40:0a:12
						UP BROADCAST MULTICAST  MTU:1500  Metric:1
						RX packets:0 errors:0 dropped:0 overruns:0 frame:0
						TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
						collisions:0 txqueuelen:1000
						RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
						  
						  
		------------ SETUP WIFI THINGS IN DEVICE ------------
		After flashing new kernel "/var/db/dhcpd.leases" won't be there so create blank dhcp_leases
		cmd --> mkdir -p /var/db
		cmd --> touch /var/db/dhcpd.leases
		cmd --> chmod 777 /var/db/dhcpd.leases
		cmd --> chown root:root /var/db/dhcpd.leases

		After flashing new kernel "/sbin/dhclient-script" wont be there so create it and fill using
		cmd --> vi /sbin/dhclient-script
		GoTo "https://github.com/mattwire/embedded_debian/blob/master/fs/sbin/dhclient-script" and download the file
		(or)
		GoTo "/home/sanjay/IMX6ul-y2_with_RTL8188EUS" in 192.168.10.193 server you will find "dhclient-script" there
		NOTE : Be sure that the file's line feed mode is in <UNIX(LF)> mode.

		cmd --> export PATH=$PATH:/home/root/dhcp_build_imx/sbin
		if you not included dhcp exe file in File system then you shoul put it in root dir using ftp the folder path is 
		/home/sanjay/CC/cross_compilation_imx/dhcp/dhcp-4.3.0b1/dhcp_build_imx
		(or)
		/home/sanjay/IMX6ul-y2_with_RTL8188EUS/dhcp_build_imx
		cmd --> chmod +x /sbin/dhclient-script				
						
		------------ CONNECT TO WIFI SERVER(HOTSPOT) ----------
		NOTE: This wifi client can only be connected with wifi server(HOTSPOT) which is having <WPA2-Personal> Security.
		cmd --> killall wpa_supplicant 
				If already connected to any other hotspot disconnect it
		cmd --> killall dhclient
				If already connected to any other hotspot disconnect it
		cmd --> ifconfig wlan0 down
		cmd --> rm -f /var/run/wpa_supplicant/wlan0
		cmd --> ifconfig wlan0 up
		cmd --> iwlist wlan0 scan
				Scanning for available networks
		cmd --> mkdir -p /etc/wpa_supplicant
		cmd --> echo -e "ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev\nupdate_config=1\nnetwork={\nssid=\"$SSID\"\npsk=\"$PASSWORD\"\n}" | tee /etc/wpa_supplicant/wpa_supplicant.conf
		cmd --> wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
		cmd --> sleep 5
		cmd --> wpa_cli status
		cmd --> ip route del default
		cmd --> dhclient -v wlan0


		--------------- CHECK WIFI CONNECTION STATUS -------------
		cmd --> iwconfig wlan0
			You will get the below thing if it is connected to hotspot
				wlan0     IEEE 802.11bgn  ESSID:"sanjay_mob"
				Mode:Managed  Frequency:2.412 GHz  Access Point: 5E:79:3F:39:62:A7
				Bit Rate=7.2 Mb/s   Tx-Power=20 dBm
				Retry short limit:7   RTS thr=2347 B   Fragment thr:off
				Encryption key:off
				Power Management:off
				Link Quality=70/70  Signal level=-22 dBm
				Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
				Tx excessive retries:0  Invalid misc:0   Missed beacon:0
		cmd --> ifconfig wlan0
			check if the ipv4 address is assigned if it not assigned then there is a problem in dhclient
			below is the example
				wlan0     Link encap:Ethernet  HWaddr 00:1d:43:40:0a:12
				inet addr:192.168.47.188  Bcast:192.168.47.255  Mask:255.255.255.0
				inet6 addr: fe80::21d:43ff:fe40:a12/64 Scope:Link
				UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
				RX packets:14 errors:0 dropped:0 overruns:0 frame:0
				TX packets:41 errors:0 dropped:0 overruns:0 carrier:0
				collisions:0 txqueuelen:1000
				RX bytes:2062 (2.0 KiB)  TX bytes:9461 (9.2 KiB)
		CMD --> echo -e "nameserver 8.8.8.8" | tee /etc/resolv.conf
		cmd --> ping 8.8.8.8

	
	RTL8731BU WIFI Module Setup Manually (for wifi connection)
	----------------
		------------ CHECK WIFI MODULE HARDWARE CONNECTION STATUS VIA USB ------------
		cmd --> lsusb
		eg:  you will get the below lines :
			Bus 002 Device 001: ID 1d6b:0002
			Bus 001 Device 002: ID 0bda:b733
			Bus 001 Device 001: ID 1d6b:0002
		
		cmd --> ifconfig -a
			eg: you will find the below lines
				wlan0     Link encap:Ethernet  HWaddr 54:f2:9f:50:29:6f
				BROADCAST MULTICAST  MTU:1500  Metric:1
				RX packets:0 errors:0 dropped:0 overruns:0 frame:0
				TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
				collisions:0 txqueuelen:1000
				RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
				
		------------ SETUP WIFI THINGS IN DEVICE ------------
		1) tranfer "dnsmasq" , "dhcpd" , "dhclient" , "hostapd" , "hostapd_cli", "dhcrelay" to the IMX device's "/bin" folder
		2) tranfer "libssl.so.1.1" "libcrypto.so.1.1" "libnl-3.so.200" to the IMX device's "/lib" folder
		3) tranfer  "libnl-3.so.200" to the IMX device's "/usr/lib" folder
		4) give permission to those files
		5) transfer "rtl8731bu_driver.ko" file to IMX device's "/lib/modules/4.1.15-1.2.0+g439d301/kernel/drivers/net/usb" folder
		
		Enabling the driver
			cmd --> insmod /lib/modules/4.1.15-1.2.0+g439d301/kernel/drivers/net/usb/rtl8731bu_driver.ko
		
		Setting dependicies for dhcp
			cmd --> mkdir -p /var/db
			cmd --> touch /var/db/dhcpd.leases
			cmd --> chmod 777 /var/db/dhcpd.leases
			cmd --> chown root:root /var/db/dhcpd.leases
			After flashing new kernel "/sbin/dhclient-script" wont be there so create it and fill using
			cmd --> vi /sbin/dhclient-script
			GoTo "https://github.com/mattwire/embedded_debian/blob/master/fs/sbin/dhclient-script" and download the file
			(or)
			GoTo "/home/sanjay/IMX6ul-y2_with_RTL8188EUS" in 192.168.10.193 server you will find "dhclient-script" there
			NOTE : Be sure that the file's line feed mode is in <UNIX(LF)> mode.
			cmd --> chmod +x /sbin/dhclient-script
		
		Enabling STA mode
			cmd --> export SSID="sanjay_mob"
			cmd --> export PASSWORD="12345678"
			cmd --> killall wpa_supplicant
			cmd --> killall dhclient
			cmd --> ifconfig wlan0 down
			cmd --> rm -f /var/run/wpa_supplicant/wlan0
			cmd --> ifconfig wlan0 up
			cmd --> iwlist wlan0 scan
			cmd --> mkdir -p /etc/wpa_supplicant
			cmd --> echo -e "ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev\nupdate_config=1\nnetwork={\nssid=\"$SSID\"\npsk=\"$PASSWORD\"\n}" | tee /etc/wpa_supplicant/wpa_supplicant.conf
			cmd --> wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf -D wext
			cmd --> sleep 5
			cmd --> wpa_cli status
			cmd --> ip route del default
			cmd --> dhclient -v wlan0
			
			after this you want to check if the ip for wlan0 is assigned 
			cmd --> ifconfig wlan0
					
					root@myd-y6ull14x14:~# ifconfig wlan0
						wlan0     Link encap:Ethernet  HWaddr 54:f2:9f:50:29:6f
								  inet addr:192.168.184.205  Bcast:192.168.184.255  Mask:255.255.255.0
								  inet6 addr: fe80::56f2:9fff:fe50:296f/64 Scope:Link
								  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
								  RX packets:28 errors:0 dropped:0 overruns:0 frame:0
								  TX packets:48 errors:0 dropped:3 overruns:0 carrier:0
								  collisions:0 txqueuelen:1000
								  RX bytes:5061 (4.9 KiB)  TX bytes:11935 (11.6 KiB)
			
		Enabling AP mode
			cmd --> mkdir /etc/hostapd
			cmd --> cat > /etc/hostapd/hostapd.conf << 'EOF'
					# Basic configuration for a wireless access point
					interface=wlan0
					driver=nl80211
					ssid=My_IMX_RTL_HOTSPOT
					hw_mode=g
					channel=7
					wmm_enabled=1
					# Enable WPA2
					auth_algs=1
					wpa=2
					wpa_passphrase=12345678
					wpa_key_mgmt=WPA-PSK
					wpa_pairwise=TKIP
					rsn_pairwise=CCMP
					# Basic security settings
					macaddr_acl=0
					ignore_broadcast_ssid=0
					# 802.11n settings
					ieee80211n=1
					ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
					# Country code
					country_code=IN
					ieee80211d=1
					# Logging settings
					logger_syslog=-1
					logger_syslog_level=2
					logger_stdout=-1
					logger_stdout_level=2
					EOF
					
			# Clean up first
			cmd --> killall hostapd wpa_supplicant
			cmd --> killall dnsmasq
			cmd --> killall dhcpd
			cmd --> ifconfig wlan0 down
			
			# Set up AP mode
			cmd --> iw dev wlan0 set type ap
			cmd --> ifconfig wlan0 up 192.168.10.123 netmask 255.255.255.0
			cmd --> route del default
			cmd --> route add default gw 192.168.10.123
			
			cmd --> hostapd -ddd /etc/hostapd/hostapd.conf &
			cmd --> dnsmasq -C /etc/dnsmasq.conf
					Transfer dnsmasq.conf to /etc
					Note : In dnsmasq.conf change the dns address to the wlan0 ip address otherwise it will disconnect after connecting becoz it will try to ping that after connecting
						37) dhcp-option=6,192.168.10.1               # Set the DNS server IP (same as dnsmasq server)
						37) dhcp-option=6,192.168.10.123               # Set the DNS server IP (same as dnsmasq server)
	
	
	WIFI CONNECTION SPEED CHECKING TOOLS
	------------
		--- checking signal strength ---
			cmd --> iwconfig wlan0
			
		--- check WIFI speed ---
			1) install "ping tool" app in your mobile
				
				speed test in STA mode in IMX and AP mode in mobile
					In mobile establish a iperf server with port number 5201
					In imx give 
						upload speed cmd -->  iperf3 -c 192.168.98.28 -p 5201 
						download speed cmd -->iperf3 -c 192.168.98.28 -p 5201 -R
					
				speed test in AP mode in IMX and STA mode in mobile
					In imx give 
						establish iperf server  cmd -->  iperf3 -s
					In mobile establish a iperf client and try to connect with port number 5201
					
**************** NEW DCU KERNEL COMPILATION BEGIN ************
This is the main kernel src for the new DCU
OLD DCU
	modem : EC25
	wifi  : NOT INTEGRATED

NEW DCU
	modem : EC200
	wifi  : fn.link 6131E-U (chipset : rtl8731BU)
	
SRC DIRECTORY 
	Local PC(192.168.10.120)        : C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\DCU_WITH_EC200_n_6131EU_WIFI_MODULES
	Github repository 				: https://github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES
	SERVER LOGIN CREDIANTIALS
		User_name	: imx6ul
		Password 	: softel
	SERVER PATHS	  
		Main dir                 : /home/imx6ul/DCU_WITH_EC200_n_6131EU_WIFI_MODULES
		old DCU Linux SRC dir    : /home/imx6ul/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/old_DCU_kernel_src.7z
		latest DCU Linux SRC dir : /home/imx6ul/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux
		Toolchain dir		     : /home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin

--- IF U GOING TO START FRESHELY FOLLOW THESE STEPS (MANUALLY) ---

	step 1 : SETTING_ENV IN SERVER:
				cmd --> export ARCH=arm
				cmd --> export CROSS_COMPILE=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
				cmd --> export imxcc=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
	
	step 2 : Download the sources from the github (https://github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES)
				cmd -> git clone https://Sanjay27112000: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@github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES.git
	step 3 : Go to the directory
				cmd -> cd DCU_WITH_EC200_n_6131EU_WIFI_MODULES
	step 4 :  extract the files
				cmd -> 7z x old_DCU_kernel_src.7z
				cmd -> tar -xvf rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e.tar.gz 
	step 5 :  Copy the driver src to the kernel source
				cmd -> cp -r rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e ./MYiR-iMX-Linux/drivers/net/wireless/rtl8733bu
	step 6 :  Extract the file system
				cmd -> tar xzpf File_System.tar.gz
	step 6 :  cmd -> cd ./MYiR-iMX-Linux
	step 7 :  cmd -> export KERNEL_SRC_DIR=$(pwd)
	step 8 :  Add the driver dir in the makefile
				cmd -> echo -e "#+++++ cms_rtl873xseries_custom_driver Begin +++++\n" >> ./drivers/net/wireless/Makefile
				cmd -> echo -e "obj-$(CONFIG_RTL8733BU)           += rtl8733bu/\n" >> ./drivers/net/wireless/Makefile
				cmd -> echo -e "#+++++ cms_rtl873xseries_custom_driver End +++++\n" >> ./drivers/net/wireless/Makefile
				
	step 9 : Add the driver Kconfig in the 
				cmd -> echo -e "#+++++ cms_rtl873xseries_custom_driver Begin +++++" > ./drivers/net/wireless/temp_insert.txt
				cmd -> echo -e "source \"drivers/net/wireless/rtl8733bu/Kconfig\"" >> ./drivers/net/wireless/temp_insert.txt
				cmd -> echo -e "#+++++ cms_rtl873xseries_custom_driver End +++++" >> ./drivers/net/wireless/temp_insert.txt
				cmd -> sed -i '/endif # WLAN/r ./drivers/net/wireless/temp_insert.txt' Kconfig
				cmd -> sed -i '/endif # WLAN/i\\' ./drivers/net/wireless/Kconfig
				cmd -> rm ./drivers/net/wireless/temp_insert.txt
	step 10 : Now make the wifi driver			
				cmd --> vi drivers/net/wireless/rtl8733bu
						152 CONFIG_PLATFORM_I386_PC = y  (--)
						152 CONFIG_PLATFORM_I386_PC = n  (++)
						
						167 CONFIG_PLATFORM_FS_MX61 = n  (--)
						167 CONFIG_PLATFORM_FS_MX61 = y  (++)
						
						--- remove the below lines ---
							1629 ifeq ($(CONFIG_PLATFORM_FS_MX61), y)
							1630 EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
							1631 ARCH := arm
							1632 CROSS_COMPILE := /home/share/CusEnv/FreeScale/arm-eabi-4.4.3/bin/arm-eabi-
							1633 KSRC ?= /home/share/CusEnv/FreeScale/FS_kernel_env
							1634 endif
						--- remove the above lines ---
						
						+++ add the below line +++
							1629 ifeq ($(CONFIG_PLATFORM_FS_MX61), y)
							1630 EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN -DRTW_USE_CFG80211_STA_EVENT -DCONFIG_IOCTL_CFG80211
							1631 ARCH := $(ARCH)
							1632 CROSS_COMPILE := $(CROSS_COMPILE)
							1633 KSRC ?= $(KERNEL_SRC_DIR)
							1634 MODULE_NAME := rtl8733bu_driver
							1635 endif
						+++ add the above line +++
						NOTE:
							KSRC is a kernel source so if you are compiling this driver to some kernel don't forget to give that kernel source path
							or else it will cause kernel mismatch error
							
							In the makefile you can give the output .ko file name in "MODULE_NAME"
							
	step 11 :  Making Wifi driver 
				Enabling wifi supporting driver cfgs(cfg80211) in kernel src code
					cmd --> make menuconfig
					enable drivers cfgs(cfg80211)
						--- Networking support---
							--- Wireless ---
								<*>   cfg80211 - wireless configuration API                                                                                     
								[ ]     nl80211 testmode command (NEW)                                                                          
								[ ]     enable developer warnings (NEW)                                                                                                         
								[ ]     cfg80211 regulatory debugging (NEW)                                                                                                        
								[ ]     cfg80211 certification onus (NEW)                                                                                                           
								[*]     enable powersave by default (NEW)                                                                                                           
								[ ]     cfg80211 DebugFS entries (NEW)                                                                                                              
								[ ]     use statically compiled regulatory rules database (NEW)                                                                                     
								[ ]     cfg80211 wireless extensions compatibility (NEW)                                                                                            
								<*>   Generic IEEE 802.11 Networking Stack (mac80211)                                                                                               
								[*]   Minstrel (NEW)                                                                                                                                
								[*]     Minstrel 802.11n support (NEW)                                                                                                              
								[ ]       Minstrel 802.11ac support (NEW)                                                                                                          
								Default rate control algorithm (Minstrel)  --->                                                                                              
								[ ]   Enable mac80211 mesh networking (pre-802.11s) support (NEW)                                                                                   
								[ ]   Enable LED triggers (NEW)                                                                                                                     
								[ ]   Export mac80211 internals in DebugFS (NEW)                                                                                                    
								[ ]   Trace all mac80211 debug messages (NEW)                                                                                                       
								[ ]   Select mac80211 debugging features (NEW)  ----
					enable driver for RTL8733BU
						--- Device Drivers ---
							--- Network device support ---
								--- Wireless LAN ---
									<*>   Realtek 8733BU USB WiFi
					save the menuconfig
					
	step 12 :  Make the kernel
					cmd --> make
					
				If everything goes well, it will produce a kernel files.
				
				KERNEL IMAGE:
					In IMX the kernel image is ZIMAGE file
					zImage location --> ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux/arch/arm/boot
				
				DTB:
					myd-y6ull-gpmi-weim.dtb (FOR Y2 BASED BOARDS)
					myd-y6ul-gpmi-weim.dtb  (FOR G2 BASED BOARDS)
					dtb file location --> ~/DCU_WITH_EC200_n_6131EU_WIFI_MODULES/MYiR-iMX-Linux/arch/arm/boot/dts
					
	step 13 :  Go to the main dir 
				cmd -> cd ..
	step 14 :  Make the File system
				cmd -> cd File_System
				cmd -> make
				Tranfer the "ubi.img" to the unit and try
					
--- IF U GOING TO START FRESHELY FOLLOW THESE STEPS (AUTOMATIC) (FASTRACK MODE...) ---				
	export ARCH=arm
	export CROSS_COMPILE=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-
	export imxcc=/home/imx6ul/Toolchain/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
	git clone https://Sanjay27112000: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@github.com/CreativeMicroSystems/DCU_WITH_EC200_n_6131EU_WIFI_MODULES.git
	cd DCU_WITH_EC200_n_6131EU_WIFI_MODULES

	7z x old_DCU_kernel_src.7z
	tar xzpf File_System.tar.gz
	tar -xvf rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e.tar.gz
	cp -r rtl8733BU_WiFi_linux_v5.13.0.1-112-g10248f4f3.20230626_COEX20230616-330e ./MYiR-iMX-Linux/drivers/net/wireless/rtl8733bu

	patch -p0 < ./patches/rtl8733bu_makefile_for_IMX.patch
	patch -p0 < ./patches/wireless_makefile_for_wifi.patch
	patch -p0 < ./patches/wireless_kconfig_for_wifi.patch
	patch -p0 < ./patches/menuconfig_for_wifi_n_802-11.patch

	export KERNEL_SRC_DIR=$(pwd)

	make clean -C ./MYiR-iMX-Linux 
	make -C ./MYiR-iMX-Linux 
	make -C ./File_System/IMX_NEW_DCU_FS/

	cp ./File_System/IMX_NEW_DCU_FS/ubi.img ./Linux_4.1.15_IMX_Loading_UtilityV_A1.2_22_Jul_2024/kernel/
	cp ./MYiR-iMX-Linux/arch/arm/boot/dts/myd-y6ul-gpmi-weim.dtb ./Linux_4.1.15_IMX_Loading_UtilityV_A1.2_22_Jul_2024/kernel/
	cp ./MYiR-iMX-Linux/arch/arm/boot/dts/myd-y6ull-gpmi-weim.dtb ./Linux_4.1.15_IMX_Loading_UtilityV_A1.2_22_Jul_2024/kernel/			
	
	**** TRANSFER THE "./Linux_4.1.15_IMX_Loading_UtilityV_A1.2_22_Jul_2024" DIR TO THE PC AND LOAD THE LINUX KERNEL TO THE UNIT ****
	ERRORS FUTURE FIX:
		transfer the proper libnl-200.so file to the /usr/lib
			root@myd-y6ull14x14:~# hostapd
			hostapd: /usr/lib/libnl-3.so.200: version `libnl_3_2_27' not found (required by hostapd)
		Give proper permissons to the /etc/hostapd.conf
			root@myd-y6ull14x14:~# vsftpd
			500 OOPS: config file not owned by correct user, or not a file
**************** NEW DCU KERNEL COMPILATION END ************