------------ IOT GW RND 1 (08/11/2024)--------------
wifi interface
	AT command PM pin : 
		check status : AT+QWIFI?
		enabling 	 : AT+QWIFI=1
		disabling 	 : AT+QWIFI=0
	Result : The FC20 module's PM enable pin is not enabling and disabling continousely
***************** Testing on 16-JAN-2025 ******************
	problem is wlan ENABLE GPIO pin toggled continousely so now we given direct gpio to that pin and enabling manually
	
***************** Testing on 16-JAN-2025 ******************
	
Sim cards interface
	The sim switching pin is GPIO11
	Export GPIO
		CMD -> echo 11 > /sys/class/gpio/export
		CMD -> echo out > /sys/class/gpio/gpio11/direction
	Enabling SIM1 slot
		CMD -> echo 1 > /sys/class/gpio/gpio11/value
	Enabling SIM2 slot
		CMD -> echo 0 > /sys/class/gpio/gpio11/value
	open AT port 
		CMD -> microcom /dev/smd9
		CMD -> at
				OK
		CMD -> at+cfun=0
				OK
		CMD -> at+cfun=1
				OK
		CMD -> at+creg?
				+CREG: 0,1
		CMD -> at+cops?
				+COPS: 0,0,"IND airtel airtel",7
		CMD -> at+csq
				+CSQ: 25,99
		CMD -> at+qiact=1
				OK
		CMD -> AT+QICSGP=1,1,"airtelgprs.com","","",1
				OK
		CMD -> AT+QIACT?
				+QIACT: 1,1,1,"100.68.139.90"

				OK
		
	Result : both sims are getting registered and ppp is enabling

Sd card interface
	CMD -> df -h
		Filesystem                Size      Used Available Use% Mounted on
		ubi0:rootfs              66.0M     43.9M     22.2M  66% /
		ubi1:modem               41.0M     34.1M      6.9M  83% /firmware
		tmpfs                    64.0K      4.0K     60.0K   6% /dev
		tmpfs                    76.2M     28.0K     76.2M   0% /run
		tmpfs                    76.2M     76.0K     76.2M   0% /var/volatile
		tmpfs                    76.2M         0     76.2M   0% /media/ram
		/dev/ubi2_0              99.5M     88.0K     99.4M   0% /usrdata
		/dev/mmcblk0p1           14.8G      1.1M     14.8G   0% /media/sdcard
	Result : reading and writing is done
	
Rtc interface
	CMD -> date MMDDhhmmssYYYY
			eg : date 08161855452024
	Result : rtc is working fine

WDT interface
	Note : WDT Feeding pin gpio 42
	Testing :
		transfer the "gpio_OUT_test.sh" to the device
		CMD -> ./gpio_OUT_test.sh 42
	Result : wdt is working 
	
DO pin 
	Note : DO gpio pin 10
	Testing :
		transfer the "gpio_OUT_test.sh" to the device
		CMD -> ./gpio_OUT_test.sh 10
	Result : Relay is powering ON and OFF but LED is in issue
	
DI pin
	Note : DI gpio pin 25
	Testing :
		transfer the "gpio_IN_test.sh" to the device
		CMD -> ./gpio_IN_test.sh 10
	Result : worked but above 20 v only it is changing the value
	
I2C BUSES
	CMNDS
		CMD -> i2cdetect -y -r 2
			eg:
					0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
				00:          -- 04 05 06 07 -- -- -- -- -- -- -- --
				10: -- -- -- -- -- -- -- -- -- -- -- UU -- -- -- --
				20: 20 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
				30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
				40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
				50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
				60: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
				70: -- -- -- -- -- -- -- --
		i2cset -y 2 0x20 0xff
		i2cget -y 2 0x20
		
	INPUTS
	
		Di 
			CMD -> i2cget -y 2 0x20
			Result : If voltage given to the pin, value got as 0x7F
			Result ERROR : worked but above 20 v only it is changing the value
			
		Default switch
			CMD -> i2cget -y 2 0x20
			Result : If default switch is pressed, value got as 0xBF
			
		Wifi cfg switch
			CMD -> i2cget -y 2 0x20
			Result : If Wifi cfg switch is pressed, value got as 0xDF
		
		ALL
			If all default switch, wifi cfg pin and DI pin pressed, value got as 0x1f
	
	OUTPUTS
		CMD -> i2cset -y 2 0x20 0xff
		4 signal strength LEDs
			addressess 
			led1 : 0001 1110 => 0x1E
				   CMD -> i2cset -y 2 0x20 0x1E
			led2 : 0001 1101 => 0x1D
				   CMD -> i2cset -y 2 0x20 0x1D
			led3 : 0001 1011 => 0x1B
				   CMD -> i2cset -y 2 0x20 0x1B
			led4 : 0001 0111 => 0x17
				   CMD -> i2cset -y 2 0x20 0x17
		Result : all are worked
		1 cloud status Led
			address
			led1 : 0000 1110 => 0x0E
				CMD -> i2cset -y 2 0x20 0x0E
		Result : worked
		
Serial interface
	*Transfer serial port testing file to the device*
	RS232
		send
			CMD -> ./serPortSendHighSpeed
			Result :  working
		receive
			CMD -> ./serPortRecvHighSpeed 4 0 10
			Result :  working
	RS485
		send
			CMD -> ./serPortSendHighSpeed
			Result :  working
		receive
			CMD -> ./serPortRecvHighSpeed 4 0 10
			Result :  working
------------ IOT GW RND 1 (08/11/2024)--------------

		
-------------- SAMPLE cmnds and REFERENCE Begin --------------
/sys/class/gpio # mi
microcom   minidlnad  miniupnpd
/sys/class/gpio # microcom /dev/smd9
at
OK
at+cfun=0
OK
at
OK
at+cfun=1
OK
at+creg?
+CREG: 0,1

OK
at+cops?
+COPS: 0

OK
at+cops?
+COPS: 0

OK
at+creg?
+CREG: 0,1

OK
at+cops?
+COPS: 0

OK
at+creg?
+CREG: 0,1

OK
at+cops?
+COPS: 0,0,"IND airtel airtel",7

OK
at+csq
+CSQ: 25,99

OK
at+qiact=1
OK
at+aicsgp
ERROR
at+qicsgp=1,1,"airtelgpars⌂
ERROR
AT+QICSGP=1,1,"airtelgprs.com","","",1
OK
AT+QIACT?
+QIACT: 1,1,1,"100.68.139.90"

OK
at+qping="8.8.8.8",1,1
ERROR

AT+CGDCONT=1,"IP","internet"
AT+CGDCONT?

-------------- SAMPLE cmnds and REFERENCE End --------------

--------- ping at cmnds ------
Serial port COM13 closed
Serial port COM13 opened
at
ERROR
at+cfun=0
OK
at+cfun=1
OK

+CPIN: READY

+QUSIM: 1

+QIND: SMS DONE

+QIND: PB DONE
at+creg?
+CREG: 0,1

OK
at+cops?
+COPS: 0,0,"IND airtel airtel",7

OK
at+csq
+CSQ: 17,99

OK
AT+QICSGP=1,1,"www","","",1
OK
AT+QIACT?
OK
AT+CGDCONT=1,"IP","internet"
OK
AT+CGDCONT?
+CGDCONT: 1,"IP","internet","0.0.0.0",0,0,0,0
+CGDCONT: 2,"IP","uninet","0.0.0.0",0,0,0,0
+CGDCONT: 3,"IPV4V6","3gpp","0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0",0,0,0,0

OK
AT+QPING=1,"8.8.8.8"
OK

+QPING: 0,"8.8.8.8",32,59,255

+QPING: 0,"8.8.8.8",32,52,255

+QPING: 0,"8.8.8.8",32,39,255

+QPING: 0,"8.8.8.8",32,63,255

+QPING: 0,4,4,0,39,63,52
--------- ping at cmnds ------
