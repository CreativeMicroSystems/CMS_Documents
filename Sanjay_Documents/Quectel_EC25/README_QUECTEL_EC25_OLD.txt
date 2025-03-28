 Add user:
 --------
	sudo useradd -m quectel-ec25-1
	sudo usermod -aG sudo quectel-ec25-1
	sudo userdel -r quectel-ec25-1
 
 
 Build system Details:
 ---------------------
	 Server IP : 192.168.10.193
	 Username  : quectel-ec25
	 Password  : softel
	 Location  : 0.

Environment Setup:
------------------
1. Untar SDK (EC25EFAR06A12M4G_OCPU_20.002.20.002_SDK.tar.bz2) from DOWNLOADED_SDK directory to SDK directory.
	tar -xvf EC25EFAR06A12M4G_OCPU_20.002.20.002_SDK.tar.bz2 -C ~/SDK

2. Run the environment setup script.
	cd ~/SDK/ql-ol-sdk
	source ql-ol-crosstool/ql-ol-crosstool-env-init
	
	vi ql-ol-kernel/build/scripts/dtc/dtc-lexer.lex.c +640
	//Arun
	//YYLTYPE yylloc;
	extern YYLTYPE yylloc;
	
	
3. Add the below lines in user profile, so that it will init automatically on login
	 29 # 29/04/2023, Quectel SDK environment init
	 30 source ~/SDK/ql-ol-sdk/ql-ol-crosstool/ql-ol-crosstool-env-init
	 
	 
Ethernet Driver add in Kernel menuconfig:
-----------------------------------------

1. Enable the below drivers for Ethernet 
	cd ~/SDK/ql-ol-sdk
	make kernel_menuconfig

	Device Drivers
		-> [*] Network device support
			-> [*] Ethernet driver support
				-> [*] Qualcom devices
				-> 		[*] Qualcomm Atheros QCA7000 support
				->		[*] Qualcomm Technologies, Inc. EMAC Gigabit Ethernet support 
	
	Device Drivers
		-> [*] Network device support
			-> [*] PHY Device support and infrastructure
			-> 		[*] Drivers for Atheros AT803X PHYs
			->		[*] Drivers for QTI Atheros QCA8337 switch
	
	make
	
	Then copy the image files and flash to unit
	
DTS Files:
---------
cp ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi.org
cp ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dtsi ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dtsi.org
cd ~/
mkdir DTS
cd DTS
ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi.org mdm9607.dtsi.org 
ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dtsi.org mdm9607-mtp.dtsi.org 

ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dts mdm9607-mtp.dts 
ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dtsi mdm9607-mtp.dtsi 
ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi mdm9607.dtsi 
ln -s ../SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-pinctrl.dtsi mdm9607-pinctrl.dtsi
 
Enable SPI Multiplexed UART:
----------------------------

 1. Refer the document, PageNo51, EC25-Quecopen_Hardware_Design_V1.2.pdf Table 15: Pin Definition of UART1 Interface (Multiplexed with SPI).
 2. Enable the UART_BLSP6 in device tree.
 3. Add the below lines, in "mdm9607.dtsi".
	
	vi ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607.dtsi
	 358         /*2018-03-15, Modify by Quinn.Zhao, to support hardware flow control*/
	 359         blsp1_uart6: serial@78b4000 {
	 360                 compatible = "qcom,msm-lsuart-v14";
	 361                 reg = <0x78b4000 0x200>;
	 362                 interrupts = <0 122 0>;
	 363                 qcom,master-id = <86>;
	 364                 clock-names = "core_clk", "iface_clk";
	 365                 clocks = <&clock_gcc clk_gcc_blsp1_uart6_apps_clk>,
	 366                                 <&clock_gcc clk_gcc_blsp1_ahb_clk>;
	 367                 pinctrl-names = "sleep", "default";
	 368                 pinctrl-0 = <&blsp1_uart6_tx_sleep &blsp1_uart6_rx_sleep &blsp1_uart6_rts_sleep &blsp1_uart6_cts_sleep>;
	 369                 pinctrl-1 = <&blsp1_uart6_tx_active &blsp1_uart6_rx_active &blsp1_uart6_rts_active &blsp1_uart6_cts_active>;
	 370                 qcom,msm-bus,name = "blsp1_uart6";
	 371                 qcom,msm-bus,num-cases = <2>;
	 372                 qcom,msm-bus,num-paths = <1>;
	 373                 qcom,msm-bus,vectors-KBps =
	 374                                 <86 512 0 0>,
	 375                                 <86 512 500 800>;
	 376                 // 29/04/2023, Modify by Arun(CMS), to enable SPI Multiplexed UART
	 377                 // status = "disabled";
	 378                 status = "ok";
	 379         };
	 
4. Add the below lines, in "mdm9607-mtp.dtsi".

	vi ~/SDK/ql-ol-sdk/ql-ol-kernel/arch/arm/boot/dts/qcom/mdm9607-mtp.dtsi
	 48 */
	 49 &blsp1_uart3 {
	 50         status = "ok";
	 51 };
	 52
	 53 // 29/04/2023, Modify by Arun(CMS), to enable SPI Multiplexed UART
	 54 &blsp1_uart6 {
	 55         status = "ok";
	 56 };
	 
5. Once modified, compile by running make command in ~/SDK.

6. Compiled binaries will be located here.
	Location : /home/quectel-ec25/SDK/ql-ol-sdk/ql-ol-extsdk/tools/quectel_mkboot
	Device tree blob : mdm9607-mtp.dtb
	Kernel image     : zImage
	

PPP support in kernel:
----------------------


serial@78b4000 

Quectel Serial Ports:

/dev/ttyHSL0 => Console port

/dev/ttyHSL1 => 2nd UART(blsp1_uart6 ( serial@78b4000 )   Multiplex with SPI)

/dev/ttyHS0   => Main UART Port

vi 



SD CARD DEBUG:
--------------
/home/quectel-ec25/SDK/ql-ol-sdk/ql-ol-kernel/Documentation/devicetree/bindings/mmc/sdhci-msm.txt

root@mdm9607-perf:/dev# ls -al; ls -al | wc -l;
total 4
drwxr-xr-x    9 root     root          5300 Apr 29 05:03 .
drwxr-xr-x   25 root     root          1976 Jan  1  1970 ..
crw-rw----    1 root     root       10,  24 Jan  1  1970 android_mbim
crw-rw----    1 root     root       10,  15 Jan  1  1970 android_rndis_qc
crw-rw----    1 root     root       10,  13 Jan  1  1970 android_serial_device
crw-rw----    1 root     root      247,   3 Jan  1  1970 apr_apps2
crw-rw----    1 root     root      241,   0 Jan  1  1970 at_usb0
crw-rw----    1 root     root      241,   1 Jan  1  1970 at_usb1
crw-rw----    1 root     root       10,  17 Jan  1  1970 ccid_bulk
crw-rw----    1 root     root       10,  18 Jan  1  1970 ccid_ctrl
crw-------    1 root     root        5,   1 Jan  1  1970 console
crw-rw----    1 root     root       10,  33 Jan  1  1970 coresight-stm
crw-rw----    1 root     root       10,  34 Jan  1  1970 coresight-tmc-etf
crw-rw----    1 root     root       10,  35 Jan  1  1970 coresight-tmc-etr
crw-rw----    1 root     root      240,   0 Jan  1  1970 coresight-tmc-etr-stream
crw-rw----    1 root     root       10,  30 Jan  1  1970 cpu_dma_latency
crw-rw----    1 root     diag      246,   0 Jan  1  1970 diag
crw-rw----    1 root     root       10,  19 Jan  1  1970 dpl_ctrl
crw-rw----    1 root     root      245,   0 Jan  1  1970 dtmf_detect
crw-rw----    1 root     root       10,  12 Jan  1  1970 embms_tm_device
crw-rw-rw-    1 root     root        1,   7 Jan  1  1970 full
crw-rw----    1 uucp     183        10, 183 Jan  1  1970 hw_random
crw-rw----    1 root     root       89,   2 Jan  1  1970 i2c-2
prw-------    1 root     root             0 Jan  1  1970 initctl
drwxr-xr-x    2 root     root            60 Jan  1  1970 input
crw-rw----    1 root     root       10,  63 Jan  1  1970 ion
crw-r-----    1 root     kmem        1,   2 Jan  1  1970 kmem
crw-rw----    1 root     root        1,  11 Jan  1  1970 kmsg
srw-rw-rw-    1 root     root             0 Jan  1  1970 log
drwxr-xr-x    2 root     root           120 Jan  1  1970 logger
crw-rw----    1 root     root       10, 237 Jan  1  1970 loop-control
brw-r-----    1 root     disk        7,   0 Jan  1  1970 loop0
brw-r-----    1 root     disk        7,   1 Jan  1  1970 loop1
brw-r-----    1 root     disk        7,   2 Jan  1  1970 loop2
brw-r-----    1 root     disk        7,   3 Jan  1  1970 loop3
brw-r-----    1 root     disk        7,   4 Jan  1  1970 loop4
brw-r-----    1 root     disk        7,   5 Jan  1  1970 loop5
brw-r-----    1 root     disk        7,   6 Jan  1  1970 loop6
brw-r-----    1 root     disk        7,   7 Jan  1  1970 loop7
-rw-r--r--    1 root     root             4 Apr 29 05:03 mdev.seq
crw-r-----    1 root     kmem        1,   1 Jan  1  1970 mem
crw-rw----    1 root     root       10,  27 Jan  1  1970 memory_bandwidth
brw-rw----    1 root     disk      179,   0 Apr 29 05:03 mmcblk0
brw-rw----    1 root     disk      179,   1 Apr 29 05:03 mmcblk0p1
crw-rw----    1 root     root      256,   0 Jan  1  1970 msm-rng
crw-rw----    1 root     root       10,  53 Jan  1  1970 msm_aac
crw-rw----    1 root     root       10,  59 Jan  1  1970 msm_aac_in
crw-rw----    1 root     root       10,  51 Jan  1  1970 msm_alac
crw-rw----    1 root     root       10,  48 Jan  1  1970 msm_amrnb
crw-rw----    1 root     root       10,  56 Jan  1  1970 msm_amrnb_in
crw-rw----    1 root     root       10,  47 Jan  1  1970 msm_amrwb
crw-rw----    1 root     root       10,  43 Jan  1  1970 msm_amrwb_in
crw-rw----    1 root     root       10,  46 Jan  1  1970 msm_amrwbplus
crw-rw----    1 root     root       10,  50 Jan  1  1970 msm_ape
crw-rw----    1 root     root       10,  62 Jan  1  1970 msm_audio_cal
crw-rw----    1 root     root       10,  45 Jan  1  1970 msm_evrc
crw-rw----    1 root     root       10,  57 Jan  1  1970 msm_evrc_in
crw-rw----    1 root     root       10,  42 Jan  1  1970 msm_hweffects
crw-rw----    1 root     root       10,  49 Jan  1  1970 msm_mp3
crw-rw----    1 root     root       10,  52 Jan  1  1970 msm_multi_aac
crw-rw----    1 root     root       10,  44 Jan  1  1970 msm_qcelp
crw-rw----    1 root     root       10,  58 Jan  1  1970 msm_qcelp_in
crw-rw----    1 root     root       10,  32 Jan  1  1970 msm_rtac
crw-rw----    1 root     root      254,   0 Jan  1  1970 msm_sps
crw-rw----    1 root     root      253,   0 Jan  1  1970 msm_thermal_query
crw-rw----    1 root     root       10,  55 Jan  1  1970 msm_wma
crw-rw----    1 root     root       10,  54 Jan  1  1970 msm_wmapro
lrwxrwxrwx    1 root     root            12 Jan  1  1970 mtab -> /proc/mounts
crw-rw----    1 root     root       90,   0 Jan  1  1970 mtd0
crw-rw----    1 root     root       90,   1 Jan  1  1970 mtd0ro
crw-rw----    1 root     root       90,   2 Jan  1  1970 mtd1
crw-rw----    1 root     root       90,  20 Jan  1  1970 mtd10
crw-rw----    1 root     root       90,  21 Jan  1  1970 mtd10ro
crw-rw----    1 root     root       90,  22 Jan  1  1970 mtd11
crw-rw----    1 root     root       90,  23 Jan  1  1970 mtd11ro
crw-rw----    1 root     root       90,  24 Jan  1  1970 mtd12
crw-rw----    1 root     root       90,  25 Jan  1  1970 mtd12ro
crw-rw----    1 root     root       90,  26 Jan  1  1970 mtd13
crw-rw----    1 root     root       90,  27 Jan  1  1970 mtd13ro
crw-rw----    1 root     root       90,  28 Jan  1  1970 mtd14
crw-rw----    1 root     root       90,  29 Jan  1  1970 mtd14ro
crw-rw----    1 root     root       90,  30 Jan  1  1970 mtd15
crw-rw----    1 root     root       90,  31 Jan  1  1970 mtd15ro
crw-rw----    1 root     root       90,  32 Jan  1  1970 mtd16
crw-rw----    1 root     root       90,  33 Jan  1  1970 mtd16ro
crw-rw----    1 root     root       90,  34 Jan  1  1970 mtd17
crw-rw----    1 root     root       90,  35 Jan  1  1970 mtd17ro
crw-rw----    1 root     root       90,  36 Jan  1  1970 mtd18
crw-rw----    1 root     root       90,  37 Jan  1  1970 mtd18ro
crw-rw----    1 root     root       90,  38 Jan  1  1970 mtd19
crw-rw----    1 root     root       90,  39 Jan  1  1970 mtd19ro
crw-rw----    1 root     root       90,   3 Jan  1  1970 mtd1ro
crw-rw----    1 root     root       90,   4 Jan  1  1970 mtd2
crw-rw----    1 root     root       90,  40 Jan  1  1970 mtd20
crw-rw----    1 root     root       90,  41 Jan  1  1970 mtd20ro
crw-rw----    1 root     root       90,   5 Jan  1  1970 mtd2ro
crw-rw----    1 root     root       90,   6 Jan  1  1970 mtd3
crw-rw----    1 root     root       90,   7 Jan  1  1970 mtd3ro
crw-rw----    1 root     root       90,   8 Jan  1  1970 mtd4
crw-rw----    1 root     root       90,   9 Jan  1  1970 mtd4ro
crw-rw----    1 root     root       90,  10 Jan  1  1970 mtd5
crw-rw----    1 root     root       90,  11 Jan  1  1970 mtd5ro
crw-rw----    1 root     root       90,  12 Jan  1  1970 mtd6
crw-rw----    1 root     root       90,  13 Jan  1  1970 mtd6ro
crw-rw----    1 root     root       90,  14 Jan  1  1970 mtd7
crw-rw----    1 root     root       90,  15 Jan  1  1970 mtd7ro
crw-rw----    1 root     root       90,  16 Jan  1  1970 mtd8
crw-rw----    1 root     root       90,  17 Jan  1  1970 mtd8ro
crw-rw----    1 root     root       90,  18 Jan  1  1970 mtd9
crw-rw----    1 root     root       90,  19 Jan  1  1970 mtd9ro
brw-rw----    1 root     root       31,   0 Jan  1  1970 mtdblock0
brw-rw----    1 root     root       31,   1 Jan  1  1970 mtdblock1
brw-rw----    1 root     root       31,  10 Jan  1  1970 mtdblock10
brw-rw----    1 root     root       31,  11 Jan  1  1970 mtdblock11
brw-rw----    1 root     root       31,  12 Jan  1  1970 mtdblock12
brw-rw----    1 root     root       31,  13 Jan  1  1970 mtdblock13
brw-rw----    1 root     root       31,  14 Jan  1  1970 mtdblock14
brw-rw----    1 root     root       31,  15 Jan  1  1970 mtdblock15
brw-rw----    1 root     root       31,  16 Jan  1  1970 mtdblock16
brw-rw----    1 root     root       31,  17 Jan  1  1970 mtdblock17
brw-rw----    1 root     root       31,  18 Jan  1  1970 mtdblock18
brw-rw----    1 root     root       31,  19 Jan  1  1970 mtdblock19
brw-rw----    1 root     root       31,   2 Jan  1  1970 mtdblock2
brw-rw----    1 root     root       31,  20 Jan  1  1970 mtdblock20
brw-rw----    1 root     root       31,   3 Jan  1  1970 mtdblock3
brw-rw----    1 root     root       31,   4 Jan  1  1970 mtdblock4
brw-rw----    1 root     root       31,   5 Jan  1  1970 mtdblock5
brw-rw----    1 root     root       31,   6 Jan  1  1970 mtdblock6
brw-rw----    1 root     root       31,   7 Jan  1  1970 mtdblock7
brw-rw----    1 root     root       31,   8 Jan  1  1970 mtdblock8
brw-rw----    1 root     root       31,   9 Jan  1  1970 mtdblock9
crw-rw----    1 root     root       10,  16 Jan  1  1970 mtp_usb
crw-rw----    1 root     root       10,  29 Jan  1  1970 network_latency
crw-rw----    1 root     root       10,  28 Jan  1  1970 network_throughput
crw-rw-rw-    1 root     root        1,   3 Jan  1  1970 null
crw-rw----    1 root     root      108,   0 Jan  1  1970 ppp
crw-rw-rw-    1 root     tty         5,   2 Jan  1  1970 ptmx
drwxr-xr-x    2 root     root             0 Jan  1  1970 pts
crw-rw----    1 root     root       10,  41 Jan  1  1970 qce
brw-r-----    1 root     disk        1,   0 Jan  1  1970 ram0
brw-r-----    1 root     disk        1,   1 Jan  1  1970 ram1
brw-r-----    1 root     disk        1,  10 Jan  1  1970 ram10
brw-r-----    1 root     disk        1,  11 Jan  1  1970 ram11
brw-r-----    1 root     disk        1,  12 Jan  1  1970 ram12
brw-r-----    1 root     disk        1,  13 Jan  1  1970 ram13
brw-r-----    1 root     disk        1,  14 Jan  1  1970 ram14
brw-r-----    1 root     disk        1,  15 Jan  1  1970 ram15
brw-r-----    1 root     disk        1,   2 Jan  1  1970 ram2
brw-r-----    1 root     disk        1,   3 Jan  1  1970 ram3
brw-r-----    1 root     disk        1,   4 Jan  1  1970 ram4
brw-r-----    1 root     disk        1,   5 Jan  1  1970 ram5
brw-r-----    1 root     disk        1,   6 Jan  1  1970 ram6
brw-r-----    1 root     disk        1,   7 Jan  1  1970 ram7
brw-r-----    1 root     disk        1,   8 Jan  1  1970 ram8
brw-r-----    1 root     disk        1,   9 Jan  1  1970 ram9
crw-rw----    1 root     root       10,  40 Jan  1  1970 ramdump_AR6320
crw-rw----    1 root     root       10,  31 Jan  1  1970 ramdump_modem
crw-rw----    1 root     root       10,  26 Jan  1  1970 ramdump_smem
crw-rw-rw-    1 root     root        1,   8 Jan  1  1970 random
prw--w----    1 root     rebooter         0 Apr 28 10:44 rebooterdev
crw-rw----    1 root     root       10,  61 Jan  1  1970 rfkill
crw-rw----    1 root     root       10,  23 Jan  1  1970 rmnet_ctrl
crw-rw----    1 root     root       10,  22 Jan  1  1970 rmnet_ctrl1
crw-rw----    1 root     root       10,  21 Jan  1  1970 rmnet_ctrl2
crw-rw----    1 root     root       10,  20 Jan  1  1970 rmnet_ctrl3
crw-rw----    1 root     root      252,   0 Jan  1  1970 rtc0
drwxrwxrwx    2 root     root            40 Jan  1  1970 shm
crw-rw----    1 root     root      248,  21 Apr 28 10:44 smd21
crw-rw----    1 root     root      247,   1 Jan  1  1970 smd22
crw-rw----    1 root     root      248,  36 Jan  1  1970 smd36
crw-rw----    1 root     root      248,   7 Jan  1  1970 smd7
crw-rw----    1 root     root      248,   8 Jan  1  1970 smd8
crw-rw----    1 root     root      248,   9 Jan  1  1970 smd9
crw-rw----    1 root     root      247,   4 Jan  1  1970 smd_pkt_loopback
crw-rw----    1 root     root      247,   0 Jan  1  1970 smdcntl0
crw-rw----    1 root     root      247,   2 Jan  1  1970 smdcntl8
crw-rw----    1 root     root       10,  60 Jan  1  1970 smem_log
drwxr-xr-x    2 root     root          1040 Apr 28 10:43 snd
drw-r--r--    2 root     root            60 Apr 28 10:43 socket
crw-rw----    1 root     root      244,   0 Jan  1  1970 subsys_AR6320
crw-rw----    1 root     root      244,   1 Jan  1  1970 subsys_modem
crw-rw-rw-    1 root     tty         5,   0 Apr 28 10:47 tty
crw--w----    1 root     root        4,   0 Jan  1  1970 tty0
crw--w----    1 root     root        4,   1 Jan  1  1970 tty1
crw--w----    1 root     root        4,  10 Jan  1  1970 tty10
crw--w----    1 root     root        4,  11 Jan  1  1970 tty11
crw--w----    1 root     root        4,  12 Jan  1  1970 tty12
crw--w----    1 root     root        4,  13 Jan  1  1970 tty13
crw--w----    1 root     root        4,  14 Jan  1  1970 tty14
crw--w----    1 root     root        4,  15 Jan  1  1970 tty15
crw--w----    1 root     root        4,  16 Jan  1  1970 tty16
crw--w----    1 root     root        4,  17 Jan  1  1970 tty17
crw--w----    1 root     root        4,  18 Jan  1  1970 tty18
crw--w----    1 root     root        4,  19 Jan  1  1970 tty19
crw--w----    1 root     root        4,   2 Jan  1  1970 tty2
crw--w----    1 root     root        4,  20 Jan  1  1970 tty20
crw--w----    1 root     root        4,  21 Jan  1  1970 tty21
crw--w----    1 root     root        4,  22 Jan  1  1970 tty22
crw--w----    1 root     root        4,  23 Jan  1  1970 tty23
crw--w----    1 root     root        4,  24 Jan  1  1970 tty24
crw--w----    1 root     root        4,  25 Jan  1  1970 tty25
crw--w----    1 root     root        4,  26 Jan  1  1970 tty26
crw--w----    1 root     root        4,  27 Jan  1  1970 tty27
crw--w----    1 root     root        4,  28 Jan  1  1970 tty28
crw--w----    1 root     root        4,  29 Jan  1  1970 tty29
crw--w----    1 root     root        4,   3 Jan  1  1970 tty3
crw--w----    1 root     root        4,  30 Jan  1  1970 tty30
crw--w----    1 root     root        4,  31 Jan  1  1970 tty31
crw--w----    1 root     root        4,  32 Jan  1  1970 tty32
crw--w----    1 root     root        4,  33 Jan  1  1970 tty33
crw--w----    1 root     root        4,  34 Jan  1  1970 tty34
crw--w----    1 root     root        4,  35 Jan  1  1970 tty35
crw--w----    1 root     root        4,  36 Jan  1  1970 tty36
crw--w----    1 root     root        4,  37 Jan  1  1970 tty37
crw--w----    1 root     root        4,  38 Jan  1  1970 tty38
crw--w----    1 root     root        4,  39 Jan  1  1970 tty39
crw--w----    1 root     root        4,   4 Jan  1  1970 tty4
crw--w----    1 root     root        4,  40 Jan  1  1970 tty40
crw--w----    1 root     root        4,  41 Jan  1  1970 tty41
crw--w----    1 root     root        4,  42 Jan  1  1970 tty42
crw--w----    1 root     root        4,  43 Jan  1  1970 tty43
crw--w----    1 root     root        4,  44 Jan  1  1970 tty44
crw--w----    1 root     root        4,  45 Jan  1  1970 tty45
crw--w----    1 root     root        4,  46 Jan  1  1970 tty46
crw--w----    1 root     root        4,  47 Jan  1  1970 tty47
crw--w----    1 root     root        4,  48 Jan  1  1970 tty48
crw--w----    1 root     root        4,  49 Jan  1  1970 tty49
crw--w----    1 root     root        4,   5 Jan  1  1970 tty5
crw--w----    1 root     root        4,  50 Jan  1  1970 tty50
crw--w----    1 root     root        4,  51 Jan  1  1970 tty51
crw--w----    1 root     root        4,  52 Jan  1  1970 tty52
crw--w----    1 root     root        4,  53 Jan  1  1970 tty53
crw--w----    1 root     root        4,  54 Jan  1  1970 tty54
crw--w----    1 root     root        4,  55 Jan  1  1970 tty55
crw--w----    1 root     root        4,  56 Jan  1  1970 tty56
crw--w----    1 root     root        4,  57 Jan  1  1970 tty57
crw--w----    1 root     root        4,  58 Jan  1  1970 tty58
crw--w----    1 root     root        4,  59 Jan  1  1970 tty59
crw--w----    1 root     root        4,   6 Jan  1  1970 tty6
crw--w----    1 root     root        4,  60 Jan  1  1970 tty60
crw--w----    1 root     root        4,  61 Jan  1  1970 tty61
crw--w----    1 root     root        4,  62 Jan  1  1970 tty62
crw--w----    1 root     root        4,  63 Jan  1  1970 tty63
crw--w----    1 root     root        4,   7 Jan  1  1970 tty7
crw--w----    1 root     root        4,   8 Jan  1  1970 tty8
crw--w----    1 root     root        4,   9 Jan  1  1970 tty9
crw--w----    1 root     root      242,   0 Jan  1  1970 ttyGS0
crw--w----    1 root     root      250,   0 Jan  1  1970 ttyHS0
crw-------    1 root     tty       249,   0 Apr 29 06:04 ttyHSL0
crw-rw----    1 root     root      239,   0 Jan  1  1970 ubi0
crw-rw----    1 root     root      239,   1 Jan  1  1970 ubi0_0
crw-rw----    1 root     root      238,   0 Jan  1  1970 ubi1
crw-rw----    1 root     root      238,   1 Jan  1  1970 ubi1_0
crw-rw----    1 root     root      237,   0 Jan  1  1970 ubi2
crw-rw----    1 root     root      237,   1 Jan  1  1970 ubi2_0
crw-rw----    1 root     root       10,  25 Jan  1  1970 ubi_ctrl
crw-rw----    1 root     root       10, 223 Jan  1  1970 uinput
crw-rw-rw-    1 root     root        1,   9 Jan  1  1970 urandom
drwxr-xr-x    3 root     root            60 Jan  1  1970 usb-ffs
crw-rw----    1 root     root       10,  14 Jan  1  1970 usb_accessory
crw-rw----    1 root     tty         7,   0 Jan  1  1970 vcs
crw-rw----    1 root     tty         7,   1 Jan  1  1970 vcs1
crw-rw----    1 root     tty         7, 128 Jan  1  1970 vcsa
crw-rw----    1 root     tty         7, 129 Jan  1  1970 vcsa1
crw-rw-rw-    1 root     root        1,   5 Jan  1  1970 zero
266
root@mdm9607-perf:/dev# uname -a
Linux mdm9607-perf 3.18.20 #12 PREEMPT Thu Apr 27 17:45:06 IST 2023 armv7l GNU/Linux
root@mdm9607-perf:/dev#


root@mdm9607-perf:/dev#  ls -al; ls -al | wc -l;
total 4
drwxr-xr-x    9 root     root          5300 Apr 29 06:26 .
drwxr-xr-x   25 root     root          1976 Jan  2  1970 ..
crw-rw----    1 root     root       10,  24 Jan  2  1970 android_mbim
crw-rw----    1 root     root       10,  15 Jan  2  1970 android_rndis_qc
crw-rw----    1 root     root       10,  13 Jan  2  1970 android_serial_device
crw-rw----    1 root     root      247,   3 Jan  2  1970 apr_apps2
crw-rw----    1 root     root      241,   0 Jan  2  1970 at_usb0
crw-rw----    1 root     root      241,   1 Jan  2  1970 at_usb1
crw-rw----    1 root     root       10,  17 Jan  2  1970 ccid_bulk
crw-rw----    1 root     root       10,  18 Jan  2  1970 ccid_ctrl
crw-------    1 root     root        5,   1 Jan  2  1970 console
crw-rw----    1 root     root       10,  33 Jan  2  1970 coresight-stm
crw-rw----    1 root     root       10,  34 Jan  2  1970 coresight-tmc-etf
crw-rw----    1 root     root       10,  35 Jan  2  1970 coresight-tmc-etr
crw-rw----    1 root     root      240,   0 Jan  2  1970 coresight-tmc-etr-stream
crw-rw----    1 root     root       10,  30 Jan  2  1970 cpu_dma_latency
crw-rw----    1 root     diag      246,   0 Jan  2  1970 diag
crw-rw----    1 root     root       10,  19 Jan  2  1970 dpl_ctrl
crw-rw----    1 root     root      245,   0 Jan  2  1970 dtmf_detect
crw-rw----    1 root     root       10,  12 Jan  2  1970 embms_tm_device
crw-rw-rw-    1 root     root        1,   7 Jan  2  1970 full
crw-rw----    1 uucp     183        10, 183 Jan  2  1970 hw_random
crw-rw----    1 root     root       89,   2 Jan  2  1970 i2c-2
prw-------    1 root     root             0 Jan  2  1970 initctl
drwxr-xr-x    2 root     root            60 Jan  2  1970 input
crw-rw----    1 root     root       10,  63 Jan  2  1970 ion
crw-r-----    1 root     kmem        1,   2 Jan  2  1970 kmem
crw-rw----    1 root     root        1,  11 Jan  2  1970 kmsg
srw-rw-rw-    1 root     root             0 Jan  2  1970 log
drwxr-xr-x    2 root     root           120 Jan  2  1970 logger
crw-rw----    1 root     root       10, 237 Jan  2  1970 loop-control
brw-r-----    1 root     disk        7,   0 Jan  2  1970 loop0
brw-r-----    1 root     disk        7,   1 Jan  2  1970 loop1
brw-r-----    1 root     disk        7,   2 Jan  2  1970 loop2
brw-r-----    1 root     disk        7,   3 Jan  2  1970 loop3
brw-r-----    1 root     disk        7,   4 Jan  2  1970 loop4
brw-r-----    1 root     disk        7,   5 Jan  2  1970 loop5
brw-r-----    1 root     disk        7,   6 Jan  2  1970 loop6
brw-r-----    1 root     disk        7,   7 Jan  2  1970 loop7
-rw-r--r--    1 root     root             4 Apr 29 06:26 mdev.seq
crw-r-----    1 root     kmem        1,   1 Jan  2  1970 mem
crw-rw----    1 root     root       10,  27 Jan  2  1970 memory_bandwidth
brw-rw----    1 root     disk      179,   0 Jan  2  1970 mmcblk0
brw-rw----    1 root     disk      179,   1 Jan  2  1970 mmcblk0p1
crw-rw----    1 root     root      256,   0 Jan  2  1970 msm-rng
crw-rw----    1 root     root       10,  53 Jan  2  1970 msm_aac
crw-rw----    1 root     root       10,  59 Jan  2  1970 msm_aac_in
crw-rw----    1 root     root       10,  51 Jan  2  1970 msm_alac
crw-rw----    1 root     root       10,  48 Jan  2  1970 msm_amrnb
crw-rw----    1 root     root       10,  56 Jan  2  1970 msm_amrnb_in
crw-rw----    1 root     root       10,  47 Jan  2  1970 msm_amrwb
crw-rw----    1 root     root       10,  43 Jan  2  1970 msm_amrwb_in
crw-rw----    1 root     root       10,  46 Jan  2  1970 msm_amrwbplus
crw-rw----    1 root     root       10,  50 Jan  2  1970 msm_ape
crw-rw----    1 root     root       10,  62 Jan  2  1970 msm_audio_cal
crw-rw----    1 root     root       10,  45 Jan  2  1970 msm_evrc
crw-rw----    1 root     root       10,  57 Jan  2  1970 msm_evrc_in
crw-rw----    1 root     root       10,  42 Jan  2  1970 msm_hweffects
crw-rw----    1 root     root       10,  49 Jan  2  1970 msm_mp3
crw-rw----    1 root     root       10,  52 Jan  2  1970 msm_multi_aac
crw-rw----    1 root     root       10,  44 Jan  2  1970 msm_qcelp
crw-rw----    1 root     root       10,  58 Jan  2  1970 msm_qcelp_in
crw-rw----    1 root     root       10,  32 Jan  2  1970 msm_rtac
crw-rw----    1 root     root      254,   0 Jan  2  1970 msm_sps
crw-rw----    1 root     root      253,   0 Jan  2  1970 msm_thermal_query
crw-rw----    1 root     root       10,  55 Jan  2  1970 msm_wma
crw-rw----    1 root     root       10,  54 Jan  2  1970 msm_wmapro
lrwxrwxrwx    1 root     root            12 Jan  2  1970 mtab -> /proc/mounts
crw-rw----    1 root     root       90,   0 Jan  2  1970 mtd0
crw-rw----    1 root     root       90,   1 Jan  2  1970 mtd0ro
crw-rw----    1 root     root       90,   2 Jan  2  1970 mtd1
crw-rw----    1 root     root       90,  20 Jan  2  1970 mtd10
crw-rw----    1 root     root       90,  21 Jan  2  1970 mtd10ro
crw-rw----    1 root     root       90,  22 Jan  2  1970 mtd11
crw-rw----    1 root     root       90,  23 Jan  2  1970 mtd11ro
crw-rw----    1 root     root       90,  24 Jan  2  1970 mtd12
crw-rw----    1 root     root       90,  25 Jan  2  1970 mtd12ro
crw-rw----    1 root     root       90,  26 Jan  2  1970 mtd13
crw-rw----    1 root     root       90,  27 Jan  2  1970 mtd13ro
crw-rw----    1 root     root       90,  28 Jan  2  1970 mtd14
crw-rw----    1 root     root       90,  29 Jan  2  1970 mtd14ro
crw-rw----    1 root     root       90,  30 Jan  2  1970 mtd15
crw-rw----    1 root     root       90,  31 Jan  2  1970 mtd15ro
crw-rw----    1 root     root       90,  32 Jan  2  1970 mtd16
crw-rw----    1 root     root       90,  33 Jan  2  1970 mtd16ro
crw-rw----    1 root     root       90,  34 Jan  2  1970 mtd17
crw-rw----    1 root     root       90,  35 Jan  2  1970 mtd17ro
crw-rw----    1 root     root       90,  36 Jan  2  1970 mtd18
crw-rw----    1 root     root       90,  37 Jan  2  1970 mtd18ro
crw-rw----    1 root     root       90,  38 Jan  2  1970 mtd19
crw-rw----    1 root     root       90,  39 Jan  2  1970 mtd19ro
crw-rw----    1 root     root       90,   3 Jan  2  1970 mtd1ro
crw-rw----    1 root     root       90,   4 Jan  2  1970 mtd2
crw-rw----    1 root     root       90,  40 Jan  2  1970 mtd20
crw-rw----    1 root     root       90,  41 Jan  2  1970 mtd20ro
crw-rw----    1 root     root       90,   5 Jan  2  1970 mtd2ro
crw-rw----    1 root     root       90,   6 Jan  2  1970 mtd3
crw-rw----    1 root     root       90,   7 Jan  2  1970 mtd3ro
crw-rw----    1 root     root       90,   8 Jan  2  1970 mtd4
crw-rw----    1 root     root       90,   9 Jan  2  1970 mtd4ro
crw-rw----    1 root     root       90,  10 Jan  2  1970 mtd5
crw-rw----    1 root     root       90,  11 Jan  2  1970 mtd5ro
crw-rw----    1 root     root       90,  12 Jan  2  1970 mtd6
crw-rw----    1 root     root       90,  13 Jan  2  1970 mtd6ro
crw-rw----    1 root     root       90,  14 Jan  2  1970 mtd7
crw-rw----    1 root     root       90,  15 Jan  2  1970 mtd7ro
crw-rw----    1 root     root       90,  16 Jan  2  1970 mtd8
crw-rw----    1 root     root       90,  17 Jan  2  1970 mtd8ro
crw-rw----    1 root     root       90,  18 Jan  2  1970 mtd9
crw-rw----    1 root     root       90,  19 Jan  2  1970 mtd9ro
brw-rw----    1 root     root       31,   0 Jan  2  1970 mtdblock0
brw-rw----    1 root     root       31,   1 Jan  2  1970 mtdblock1
brw-rw----    1 root     root       31,  10 Jan  2  1970 mtdblock10
brw-rw----    1 root     root       31,  11 Jan  2  1970 mtdblock11
brw-rw----    1 root     root       31,  12 Jan  2  1970 mtdblock12
brw-rw----    1 root     root       31,  13 Jan  2  1970 mtdblock13
brw-rw----    1 root     root       31,  14 Jan  2  1970 mtdblock14
brw-rw----    1 root     root       31,  15 Jan  2  1970 mtdblock15
brw-rw----    1 root     root       31,  16 Jan  2  1970 mtdblock16
brw-rw----    1 root     root       31,  17 Jan  2  1970 mtdblock17
brw-rw----    1 root     root       31,  18 Jan  2  1970 mtdblock18
brw-rw----    1 root     root       31,  19 Jan  2  1970 mtdblock19
brw-rw----    1 root     root       31,   2 Jan  2  1970 mtdblock2
brw-rw----    1 root     root       31,  20 Jan  2  1970 mtdblock20
brw-rw----    1 root     root       31,   3 Jan  2  1970 mtdblock3
brw-rw----    1 root     root       31,   4 Jan  2  1970 mtdblock4
brw-rw----    1 root     root       31,   5 Jan  2  1970 mtdblock5
brw-rw----    1 root     root       31,   6 Jan  2  1970 mtdblock6
brw-rw----    1 root     root       31,   7 Jan  2  1970 mtdblock7
brw-rw----    1 root     root       31,   8 Jan  2  1970 mtdblock8
brw-rw----    1 root     root       31,   9 Jan  2  1970 mtdblock9
crw-rw----    1 root     root       10,  16 Jan  2  1970 mtp_usb
crw-rw----    1 root     root       10,  29 Jan  2  1970 network_latency
crw-rw----    1 root     root       10,  28 Jan  2  1970 network_throughput
crw-rw-rw-    1 root     root        1,   3 Jan  2  1970 null
crw-rw----    1 root     root      108,   0 Jan  2  1970 ppp
crw-rw-rw-    1 root     tty         5,   2 Jan  2  1970 ptmx
drwxr-xr-x    2 root     root             0 Jan  1  1970 pts
crw-rw----    1 root     root       10,  41 Jan  2  1970 qce
brw-r-----    1 root     disk        1,   0 Jan  2  1970 ram0
brw-r-----    1 root     disk        1,   1 Jan  2  1970 ram1
brw-r-----    1 root     disk        1,  10 Jan  2  1970 ram10
brw-r-----    1 root     disk        1,  11 Jan  2  1970 ram11
brw-r-----    1 root     disk        1,  12 Jan  2  1970 ram12
brw-r-----    1 root     disk        1,  13 Jan  2  1970 ram13
brw-r-----    1 root     disk        1,  14 Jan  2  1970 ram14
brw-r-----    1 root     disk        1,  15 Jan  2  1970 ram15
brw-r-----    1 root     disk        1,   2 Jan  2  1970 ram2
brw-r-----    1 root     disk        1,   3 Jan  2  1970 ram3
brw-r-----    1 root     disk        1,   4 Jan  2  1970 ram4
brw-r-----    1 root     disk        1,   5 Jan  2  1970 ram5
brw-r-----    1 root     disk        1,   6 Jan  2  1970 ram6
brw-r-----    1 root     disk        1,   7 Jan  2  1970 ram7
brw-r-----    1 root     disk        1,   8 Jan  2  1970 ram8
brw-r-----    1 root     disk        1,   9 Jan  2  1970 ram9
crw-rw----    1 root     root       10,  40 Jan  2  1970 ramdump_AR6320
crw-rw----    1 root     root       10,  31 Jan  2  1970 ramdump_modem
crw-rw----    1 root     root       10,  26 Jan  2  1970 ramdump_smem
crw-rw-rw-    1 root     root        1,   8 Jan  2  1970 random
prw--w----    1 root     rebooter         0 Apr 29 06:26 rebooterdev
crw-rw----    1 root     root       10,  61 Jan  2  1970 rfkill
crw-rw----    1 root     root       10,  23 Jan  2  1970 rmnet_ctrl
crw-rw----    1 root     root       10,  22 Jan  2  1970 rmnet_ctrl1
crw-rw----    1 root     root       10,  21 Jan  2  1970 rmnet_ctrl2
crw-rw----    1 root     root       10,  20 Jan  2  1970 rmnet_ctrl3
crw-rw----    1 root     root      252,   0 Jan  2  1970 rtc0
drwxrwxrwx    2 root     root            40 Jan  2  1970 shm
crw-rw----    1 root     root      248,  21 Apr 29 06:26 smd21
crw-rw----    1 root     root      247,   1 Jan  2  1970 smd22
crw-rw----    1 root     root      248,  36 Jan  2  1970 smd36
crw-rw----    1 root     root      248,   7 Jan  2  1970 smd7
crw-rw----    1 root     root      248,   8 Jan  2  1970 smd8
crw-rw----    1 root     root      248,   9 Jan  2  1970 smd9
crw-rw----    1 root     root      247,   4 Jan  2  1970 smd_pkt_loopback
crw-rw----    1 root     root      247,   0 Jan  2  1970 smdcntl0
crw-rw----    1 root     root      247,   2 Jan  2  1970 smdcntl8
crw-rw----    1 root     root       10,  60 Jan  2  1970 smem_log
drwxr-xr-x    2 root     root          1040 Apr 29 06:26 snd
drw-r--r--    2 root     root            60 Apr 29 06:26 socket
crw-rw----    1 root     root      244,   0 Jan  2  1970 subsys_AR6320
crw-rw----    1 root     root      244,   1 Jan  2  1970 subsys_modem
crw-rw-rw-    1 root     tty         5,   0 Apr 29 06:27 tty
crw--w----    1 root     root        4,   0 Jan  2  1970 tty0
crw--w----    1 root     root        4,   1 Jan  2  1970 tty1
crw--w----    1 root     root        4,  10 Jan  2  1970 tty10
crw--w----    1 root     root        4,  11 Jan  2  1970 tty11
crw--w----    1 root     root        4,  12 Jan  2  1970 tty12
crw--w----    1 root     root        4,  13 Jan  2  1970 tty13
crw--w----    1 root     root        4,  14 Jan  2  1970 tty14
crw--w----    1 root     root        4,  15 Jan  2  1970 tty15
crw--w----    1 root     root        4,  16 Jan  2  1970 tty16
crw--w----    1 root     root        4,  17 Jan  2  1970 tty17
crw--w----    1 root     root        4,  18 Jan  2  1970 tty18
crw--w----    1 root     root        4,  19 Jan  2  1970 tty19
crw--w----    1 root     root        4,   2 Jan  2  1970 tty2
crw--w----    1 root     root        4,  20 Jan  2  1970 tty20
crw--w----    1 root     root        4,  21 Jan  2  1970 tty21
crw--w----    1 root     root        4,  22 Jan  2  1970 tty22
crw--w----    1 root     root        4,  23 Jan  2  1970 tty23
crw--w----    1 root     root        4,  24 Jan  2  1970 tty24
crw--w----    1 root     root        4,  25 Jan  2  1970 tty25
crw--w----    1 root     root        4,  26 Jan  2  1970 tty26
crw--w----    1 root     root        4,  27 Jan  2  1970 tty27
crw--w----    1 root     root        4,  28 Jan  2  1970 tty28
crw--w----    1 root     root        4,  29 Jan  2  1970 tty29
crw--w----    1 root     root        4,   3 Jan  2  1970 tty3
crw--w----    1 root     root        4,  30 Jan  2  1970 tty30
crw--w----    1 root     root        4,  31 Jan  2  1970 tty31
crw--w----    1 root     root        4,  32 Jan  2  1970 tty32
crw--w----    1 root     root        4,  33 Jan  2  1970 tty33
crw--w----    1 root     root        4,  34 Jan  2  1970 tty34
crw--w----    1 root     root        4,  35 Jan  2  1970 tty35
crw--w----    1 root     root        4,  36 Jan  2  1970 tty36
crw--w----    1 root     root        4,  37 Jan  2  1970 tty37
crw--w----    1 root     root        4,  38 Jan  2  1970 tty38
crw--w----    1 root     root        4,  39 Jan  2  1970 tty39
crw--w----    1 root     root        4,   4 Jan  2  1970 tty4
crw--w----    1 root     root        4,  40 Jan  2  1970 tty40
crw--w----    1 root     root        4,  41 Jan  2  1970 tty41
crw--w----    1 root     root        4,  42 Jan  2  1970 tty42
crw--w----    1 root     root        4,  43 Jan  2  1970 tty43
crw--w----    1 root     root        4,  44 Jan  2  1970 tty44
crw--w----    1 root     root        4,  45 Jan  2  1970 tty45
crw--w----    1 root     root        4,  46 Jan  2  1970 tty46
crw--w----    1 root     root        4,  47 Jan  2  1970 tty47
crw--w----    1 root     root        4,  48 Jan  2  1970 tty48
crw--w----    1 root     root        4,  49 Jan  2  1970 tty49
crw--w----    1 root     root        4,   5 Jan  2  1970 tty5
crw--w----    1 root     root        4,  50 Jan  2  1970 tty50
crw--w----    1 root     root        4,  51 Jan  2  1970 tty51
crw--w----    1 root     root        4,  52 Jan  2  1970 tty52
crw--w----    1 root     root        4,  53 Jan  2  1970 tty53
crw--w----    1 root     root        4,  54 Jan  2  1970 tty54
crw--w----    1 root     root        4,  55 Jan  2  1970 tty55
crw--w----    1 root     root        4,  56 Jan  2  1970 tty56
crw--w----    1 root     root        4,  57 Jan  2  1970 tty57
crw--w----    1 root     root        4,  58 Jan  2  1970 tty58
crw--w----    1 root     root        4,  59 Jan  2  1970 tty59
crw--w----    1 root     root        4,   6 Jan  2  1970 tty6
crw--w----    1 root     root        4,  60 Jan  2  1970 tty60
crw--w----    1 root     root        4,  61 Jan  2  1970 tty61
crw--w----    1 root     root        4,  62 Jan  2  1970 tty62
crw--w----    1 root     root        4,  63 Jan  2  1970 tty63
crw--w----    1 root     root        4,   7 Jan  2  1970 tty7
crw--w----    1 root     root        4,   8 Jan  2  1970 tty8
crw--w----    1 root     root        4,   9 Jan  2  1970 tty9
crw--w----    1 root     root      242,   0 Jan  2  1970 ttyGS0
crw--w----    1 root     root      250,   0 Jan  2  1970 ttyHS0
crw-------    1 root     tty       249,   0 Apr 29 06:27 ttyHSL0
crw-rw----    1 root     root      239,   0 Jan  2  1970 ubi0
crw-rw----    1 root     root      239,   1 Jan  2  1970 ubi0_0
crw-rw----    1 root     root      238,   0 Jan  2  1970 ubi1
crw-rw----    1 root     root      238,   1 Jan  2  1970 ubi1_0
crw-rw----    1 root     root      237,   0 Jan  2  1970 ubi2
crw-rw----    1 root     root      237,   1 Jan  2  1970 ubi2_0
crw-rw----    1 root     root       10,  25 Jan  2  1970 ubi_ctrl
crw-rw----    1 root     root       10, 223 Jan  2  1970 uinput
crw-rw-rw-    1 root     root        1,   9 Jan  2  1970 urandom
drwxr-xr-x    3 root     root            60 Jan  2  1970 usb-ffs
crw-rw----    1 root     root       10,  14 Jan  2  1970 usb_accessory
crw-rw----    1 root     tty         7,   0 Jan  2  1970 vcs
crw-rw----    1 root     tty         7,   1 Jan  2  1970 vcs1
crw-rw----    1 root     tty         7, 128 Jan  2  1970 vcsa
crw-rw----    1 root     tty         7, 129 Jan  2  1970 vcsa1
crw-rw-rw-    1 root     root        1,   5 Jan  2  1970 zero
266
root@mdm9607-perf:/dev# 