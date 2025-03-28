------------------ QUECTEL EC25-E with QUECTEL FC20-Q73 -----------

Note : The Quectel FC20-Q73 module uses the Qualcomm QCA6174 chipset inside it.
Note : In the vendor's default Kernel the "Quectel FC20-Q73" is enabled by default.

step 1: Flash the default kernel given by vendor via QFlash

step 2: Now we want to make the WIFI connection test files given by vendor in SDK folder
		step 1: Untar SDK (EC25EFAR06A12M4G_OCPU_20.002.20.002_SDK.tar.bz2) from DOWNLOADED_SDK directory to SDK directory.
				tar -xvf EC25EFAR06A12M4G_OCPU_20.002.20.002_SDK.tar.bz2 -C ~/SDK
		step 2: Run the environment setup script.
				cd ~/SDK/ql-ol-sdk
				source ql-ol-crosstool/ql-ol-crosstool-env-init
		step 3: cd ~/SDK/ql-ol-sdk/ql-ol-extsdk/example/wifi$
		step 4: make
		step 5: Transfer "wifi_api_test" exe file to the unit 
step 3: Give permission to the file "wifi_api_test"
		chmod +x wifi_api_test
step 4: Run the file
		./wifi_api_test

If you run the "wifi_api_test" file you will get 10 cases
	/*************************/
	case 1: Configure WiFi
	case 2: Enable WiFi
	case 3: Activate Hostapd
	case 4: Activate WPA Supplicant
	case 5: Get WPA Supplicant status
	case 6: Get WiFi configuraction
	case 7: Register Event Callback
	case 8: Enable Module WiFi
	case 9: Get WiFi status
	case 10:Set WiFi Country Code
	case 11: Scan and Scan dump
	case 100: exit
	Please input case: 
	/*************************/
Below you can find for which mode what commands want to give.

------ For Connecting with Hotspot [Quectel EC25 will act as wifi client] [Mode : STA] ------
		Enabling FC20-Q73 wifi module 
			-> Please input case: 8
			-> Please input WiFi module status(1-Enable/0-Disable): 1
				If the "Quectel FC20-Q73" module is successfully connected with "QUECTEL EC25-E" then you will find this 
				"/sys/devices/7824900.sdhci/mmc_host/mmc0/mmc0:0001/mmc0:0001:1/device" path in your device after Enable Module wifi
				NOTE: If the path is not available check Hardware connection
				
		Configuring WIFI parameters
			-> Please input case: 1
			-> Please input WiFi work mode(0-STA/1-AP0/2-AP0+STA/3-AP0+AP1): 0
			-> Please input STA ssid(maximun size 32): CMS_122166189229
			-> Please input STA type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
			-> Please input STA WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 1
			-> Please input STA WPA PSK password: softel123
			
		Enabling WIFI
			-> Please input case: 2
			-> Please input WiFi status(0-Disable/1-Enable/2-Async Enable/3-Async Disable): 1
			-> Please input case: 100
		
		Testing the Connection
			-> ifconfig
				[You should get this things if the wifi Successfully connected]
				  wlan0     Link encap:Ethernet  HWaddr 74:76:5B:70:24:8C
				  inet addr:192.168.12.16  Bcast:192.168.12.255  Mask:255.255.255.0
				  inet6 addr: fe80::7676:5bff:fe70:248c/64 Scope:Link
				  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
				  RX packets:23 errors:0 dropped:0 overruns:0 frame:0
				  TX packets:13 errors:0 dropped:0 overruns:0 carrier:0
				  collisions:0 txqueuelen:3000
				  RX bytes:4021 (3.9 KiB)  TX bytes:1301 (1.2 KiB)
			-> ping 8.8.8.8
			
------ For Establishing Hotspot[2.4 GHz] [Quectel EC25 will act as HOTSPOT] [Mode : AP] ------

		Enabling FC20-Q73 wifi module 
			-> Please input case: 8
			-> Please input WiFi module status(1-Enable/0-Disable): 1
				If the "Quectel FC20-Q73" module is successfully connected with "QUECTEL EC25-E" then you will find this 
				"/sys/devices/7824900.sdhci/mmc_host/mmc0/mmc0:0001/mmc0:0001:1/device" path in your device after Enable Module wifi
				NOTE: If the path is not available check Hardware connection
				
		Configuring WIFI parameters
			-> Please input case: 1
			-> Please input WiFi work mode(0-STA/1-AP0/2-AP0+STA/3-AP0+AP1): 1
			-> Please input WiFi index 0 ssid(maximun size 32): QUEC_EC25_CMS_HOTSPOT
			-> Please input WiFi index 0 IEEE 802.11 mode(0-b/1-bg/2-bgn/3-a/4-an/5-ac): 2
			-> Please input WiFi index 0 Bandwidth(0-20MHz/1-40MHz/2-80MHz): 0
			-> Please input WiFi index 0 channel: 11
			-> Please input WiFi index 0 maximun station(1-16):5
			-> Please input WiFi index 0 type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
			-> Please input WiFi index 0 WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 0
			-> Please input WiFi index 0 WPA PSK password: 12345678

		Enabling WIFI
			-> Please input case: 2
			-> Please input WiFi status(0-Disable/1-Enable/2-Async Enable/3-Async Disable): 1
			-> Please input case: 100
		
		Testing the Connection
			-> ifconfig
				[You should get this things if the Hotspot Successfully Established]
				wlan0     Link encap:Ethernet  HWaddr 74:76:5B:70:24:8C
				inet6 addr: fe80::7676:5bff:fe70:248c/64 Scope:Link
				UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
				RX packets:0 errors:0 dropped:0 overruns:0 frame:0
				TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
				collisions:0 txqueuelen:3000
				RX bytes:0 (0.0 B)  TX bytes:752 (752.0 B)
			
------ For Connecting with Hotspot[2.4 GHz] and establishing HOTSPOT [Quectel EC25 will act as both wifi client and Hotspot] [Mode : STA+AP]------
		
		Enabling FC20-Q73 wifi module 
			-> Please input case: 8
			-> Please input WiFi module status(1-Enable/0-Disable): 1
				If the "Quectel FC20-Q73" module is successfully connected with "QUECTEL EC25-E" then you will find this 
				"/sys/devices/7824900.sdhci/mmc_host/mmc0/mmc0:0001/mmc0:0001:1/device" path in your device after Enable Module wifi
				NOTE: If the path is not available check Hardware connection
				
		Configuring parameters
			Configuring HOTSPOT parameters
				-> Please input case: 1
				-> Please input WiFi work mode(0-STA/1-AP0/2-AP0+STA/3-AP0+AP1): 2
				-> Please input WiFi index 0 ssid(maximun size 32): QUECTEL_CMS_TEST_HOTSPOT
				-> Please input WiFi index 0 IEEE 802.11 mode(0-b/1-bg/2-bgn/3-a/4-an/5-ac): 2
				-> Please input WiFi index 0 Bandwidth(0-20MHz/1-40MHz/2-80MHz): 0
				-> Please input WiFi index 0 channel: 11
				-> Please input WiFi index 0 maximun station(1-16):5
				-> Please input WiFi index 0 type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
				-> Please input WiFi index 0 WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 0
				-> Please input WiFi index 0 WPA PSK password: 12345678
			Configuring WIFI parameters
				-> Please input STA ssid(maximun size 32): CMS_122166189229
				-> Please input STA type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
				-> Please input STA WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 1
				-> Please input STA WPA PSK password: softel123
		
		Enabling WIFI
			-> Please input case: 2
			-> Please input WiFi status(0-Disable/1-Enable/2-Async Enable/3-Async Disable): 1
			-> Please input case: 100
		
		Testing the Connection
			-> ifconfig
				[You should get this things if the wifi Successfully connected and Hotspot Successfully Established]
				wlan0     Link encap:Ethernet  HWaddr 74:76:5B:70:24:8C
						  inet addr:192.168.12.16  Bcast:192.168.12.255  Mask:255.255.255.0
						  inet6 addr: fe80::7676:5bff:fe70:248c/64 Scope:Link
						  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
						  RX packets:533 errors:0 dropped:0 overruns:0 frame:0
						  TX packets:182 errors:0 dropped:0 overruns:0 carrier:0
						  collisions:0 txqueuelen:3000
						  RX bytes:76786 (74.9 KiB)  TX bytes:11857 (11.5 KiB)

				wlan1     Link encap:Ethernet  HWaddr 02:03:7F:D6:00:01
						  inet6 addr: fe80::3:7fff:fed6:1/64 Scope:Link
						  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
						  RX packets:0 errors:0 dropped:0 overruns:0 frame:0
						  TX packets:69 errors:0 dropped:8 overruns:0 carrier:0
						  collisions:0 txqueuelen:3000
						  RX bytes:0 (0.0 B)  TX bytes:3976 (3.8 KiB)
			-> ping 8.8.8.8

------ For establishing HOTSPOT[2.4GHz & 5 GHz] [Quectel EC25 will act as hotspot] [Mode : AP+AP]------
		
		Enabling FC20-Q73 wifi module 
			-> Please input case: 8
			-> Please input WiFi module status(1-Enable/0-Disable): 1
				If the "Quectel FC20-Q73" module is successfully connected with "QUECTEL EC25-E" then you will find this 
				"/sys/devices/7824900.sdhci/mmc_host/mmc0/mmc0:0001/mmc0:0001:1/device" path in your device after Enable Module wifi
				NOTE: If the path is not available check Hardware connection
				
		Configuring parameters
			Configuring HOTSPOT 2.4GHz parameters
				-> Please input case: 1
				-> Please input WiFi work mode(0-STA/1-AP0/2-AP0+STA/3-AP0+AP1): 3
				-> Please input WiFi index 0 ssid(maximun size 32): QUECTEL_CMS_TEST_HOTSPOT_2.4GHZ
				-> Please input WiFi index 0 IEEE 802.11 mode(0-b/1-bg/2-bgn/3-a/4-an/5-ac): 2
				-> Please input WiFi index 0 Bandwidth(0-20MHz/1-40MHz/2-80MHz): 0
				-> Please input WiFi index 0 channel: 11
				-> Please input WiFi index 0 maximun station(1-16):5
				-> Please input WiFi index 0 type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
				-> Please input WiFi index 0 WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 0
				-> Please input WiFi index 0 WPA PSK password: 12345678
			
			Configuring HOTSPOT 5GHz parameters
				-> Please input WiFi index 1 ssid(maximun size 32): QUECTEL_CMS_TEST_HOTSPOT_5GHZ
				-> Please input WiFi index 1 IEEE 802.11 mode(0-b/1-bg/2-bgn/3-a/4-an/5-ac): 4
				-> Please input WiFi index 1 Bandwidth(0-20MHz/1-40MHz/2-80MHz): 0
				-> Please input WiFi index 1 channel: 149
				-> Please input WiFi index 1 maximun station(1-16):5
				-> Please input WiFi index 1 type of authentication(0-OPEN/1-WPA PSK/2-WPA2 PSK): 2
				-> Please input WiFi index 1 WPA PSK pairwise(0-AUTH/1-TKIP/2-AES): 0
				-> Please input WiFi index 1 WPA PSK password: 12345678
			
		Enabling WIFI
			-> Please input case: 2
			-> Please input WiFi status(0-Disable/1-Enable/2-Async Enable/3-Async Disable): 1
			-> Please input case: 100

		Testing the Connection
			-> ifconfig
				[You should get this things if the wifi Successfully connected]
				wlan0     Link encap:Ethernet  HWaddr 74:76:5B:70:24:8C
						  inet6 addr: fe80::7676:5bff:fe70:248c/64 Scope:Link
						  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
						  RX packets:495 errors:0 dropped:0 overruns:0 frame:0
						  TX packets:383 errors:0 dropped:1 overruns:0 carrier:0
						  collisions:0 txqueuelen:3000
						  RX bytes:52856 (51.6 KiB)  TX bytes:30620 (29.9 KiB)

				wlan1     Link encap:Ethernet  HWaddr 02:03:7F:D6:00:01
						  inet6 addr: fe80::3:7fff:fed6:1/64 Scope:Link
						  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
						  RX packets:799 errors:0 dropped:0 overruns:0 frame:0
						  TX packets:466 errors:0 dropped:1 overruns:0 carrier:0
						  collisions:0 txqueuelen:3000
						  RX bytes:99763 (97.4 KiB)  TX bytes:40039 (39.1 KiB)

-------------------------- CROSS COMPILING JSON-C ----------------------
Step 1: Download the Library
		You can download the latest release of JSON-C from the official GitHub repository or the provided links. For version 0.17, use the following command:
		text
		-> wget https://s3.amazonaws.com/json-c_releases/releases/json-c-0.17.tar.gz
		
Step 2: Build the Library
		Extract the downloaded tarball (if you downloaded it):
		text
		-> tar -xzf json-c-0.17.tar.gz
		-> cd json-c-0.17

Step 3: Create a build directory:
		text
		-> mkdir build
		-> cd build	

Step 4: Run CMake:
		text
		-> cmake -DCMAKE_INSTALL_PREFIX=/usr \
		   -DCMAKE_BUILD_TYPE=Release \
		   -DCMAKE_C_COMPILER=/home/sanjay/KERNEL/quectel/SDK/ql-ol-sdk/ql-ol-crosstool/sysroots/x86_64-oesdk-linux/usr/bin/arm-oe-linux-gnueabi/arm-oe-linux-gnueabi-gcc \
		   -DCMAKE_SYSROOT=/home/sanjay/KERNEL/quectel/SDK/ql-ol-sdk/ql-ol-crosstool/sysroots/armv7a-vfp-neon-oe-linux-gnueabi \
		   -DCMAKE_SYSTEM_NAME=Linux \
		   -DCMAKE_SYSTEM_PROCESSOR=arm ..

Step 5: Compile the library:
		text
		-> make

Step 6: Run tests (optional):
		text
		-> make test

-------------------------- CROSS COMPILING BASH ----------------------
wget https://ftp.gnu.org/gnu/bash/bash-5.2.37.tar.gz
tar -xvzf bash-5.2.37.tar.gz
cd bash-5.2.37/
export ARCH=arm
export CC=/home/sanjay/KERNEL/quectel/SDK/ql-ol-sdk/ql-ol-crosstool/sysroots/x86_64-oesdk-linux/usr/bin/arm-oe-linux-gnueabi/arm-oe-linux-gnueabi-gcc
./configure --host=arm --prefix=$(pwd)/build
mkdir build
make
mv bash build

------------- VSFTPD -----------------
cmd --> wget https://security.appspot.com/downloads/vsftpd-3.0.5.tar.gz
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
	


+++++++++++++++++ DNSMASQ NOTE ++++++++++++++++++
#PATCH TO FIX THE PINGING ISSUE WHEN DUAL AP [AP0+AP1] ESTABLISHED
If we try to enable AP_AP mode [dual AP] in code that time we will use the api's provided by vendor 
After enabling wifi the api will merge both AP+AP interfaces wlan0 and waln1 in a single bridge => bridge0 ,
	This effectively combines the two interfaces
	treating them as a single logical interface for IP assignment
	If you want wlan0 and wlan1 to operate independently, you need to Remove the bridge configuration
	remove bridge using cmd
	cmd -> brctl delif bridge0 wlan0
	cmd -> brctl delif bridge0 wlan1
	cmd -> ifconfig bridge0 down
	cmd -> brctl delbr bridge0
Aslo after establishing the bridge it will initate dnsmasq serverfor that bridge using this cmd -> 
	dnsmasq -i bridge0 -I lo -z --dhcp-range=bridge0,192.168.225.20,192.168.225.60,255.255.255.0,43200 --dhcp-hostsfile=/etc/dhcp_hosts --dhcp-option-force=6,192.168.225.1 --dhcp-option-force=120,abcd.com
	kill the process using command -> pkill dnsmasq
Set Ip seperatey for both wlan0 and wlan 1
	ifconfig wlan0 192.168.10.108
	ifconfif wlan1 192.168.20.108
	NOTE: **********SUBNET SHOULD BE UNIQUE FOR BOTH WLAN0 AND WALN1*******
Run the dnsmasq seperately for wlan0 and wlan1
	dnsmasq -i wlan0 -I lo -z --dhcp-range=192.168.10.100,192.168.10.110,255.255.255.0,43200
	dnsmasq -i wlan1 -I lo -z --dhcp-range=192.168.10.120,192.168.10.130,255.255.255.0,43200
+++++++++++++++++ DNSMASQ NOTE ++++++++++++++++++
















-> QCA6174 Driver add in Kernel menuconfig:
-> -----------------------------------------
-> Device Drivers
		-> -> [*] Network device support
-> 			-> [*]   Wireless LAN  --->  
				-> ->  <*>   Qualcomm WCNSS CORE driver                                                                                                                    x x
-> 					->  <*>     Qualcomm WCNSS Pronto Support                                                                                                               x x
					-> ->  [*]       Enable/disable WCNSS register dump when there is a WCNSS bite
				-> ->  <*>   WCNSS pre-alloc memory support
				-> ->  <*>   Enable CNSS crypto support
				-> ->  -*-   CNSS driver for wifi module
				-> ->  <*>   Flag to enable platform driver for SIDO based wifi device
				-> ->  <*>   Qualcomm CORE driver for QCA6174 with SDIO interface
				-> -> 	<*>   Qualcomm core WLAN driver for QCA6174 chipset
-> 				
-> vi ~/KERNEL/quectel/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi