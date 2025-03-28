
************************************ FLASHING UTILITY UI SETTINGS BEGIN *********************************

SPI Mode  --> DOUT //FIXED
SPI Speed --> 40MHz //FIXED
BAUD_RATE --> 921600 //fixed
uncheck DoNotChgBin option
flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin\bootloader_qio_80m.bin --> 0x1000
flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin\partition_table.bin    --> 0x8000
flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin\firmware1.0.bin        --> 0x10000
flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin\boot_app0.bin          --> 0xe000
flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin\filesystem.spiffs.bin  --> 0x290000 (For spiffs)    (If we using FATFS --> 0x291000)

************************************ FLASHING UTILITY UI SETTINGS END *********************************



NOTE : In latest utility version(3.9.5) the spi mode of QIO and the speed of 40MHz is not working fine. 
       If we flash in that mode the unit will restart continousely. So we using DOUT in 40MHz speed.
       
************************************ CREATING BIN FILES BEGIN ************************************

file_name --> bootloader.bin
bootloader creation:
In the latest arduino versions there is no bootloader.bin instead of that they gave bootloader.elf file
bootloader.elf loc --> C:\Users\Admin\Documents\ArduinoData\packages\esp32\hardware\esp32\2.0.9\tools\sdk\esp32\bin
to convert .elf to .bin file 
step 1 : Download https://github.com/espressif/esptool/archive/refs/heads/master.zip
step 2 : Open esp_tool.py 
step 3 : run this cmd --> python .\esptool.py --chip ESP32 elf2image <ELF FILE>  
						  eg : & C:/Python311/python.exe .\esptool.py --chip ESP32 elf2image C:\Users\Admin\Documents\ArduinoData\packages\esp32\hardware\esp32\2.0.9\tools\sdk\esp32\bin\bootloader_qio_80m.elf
step 4 : you will find the .bin file in the same name of.elf'
step 5 : rename it to bootloader.bin
FLASHING_ADDRESS    --> 0x1000


file_name  			--> boot_app0.bin		
file_loc   			--> C:\Users\Admin\Documents\ArduinoData\packages\esp32\hardware\esp32\2.0.9\tools\partitions
FLASHING_ADDRESS	       --> 0xe000


file_name  			--> partition.bin
step 1 : download gen_esp32part.py file in https://github.com/espressif/arduino-esp32/blob/master/tools/gen_esp32part.py
step 2 : goto C:\Users\Admin\Documents\ArduinoData\packages\esp32\hardware\esp32\2.0.9\tools\partitions
step 3 : you will find lot of .csv files choose which file you want to convert to bin
step 4 : open gen_esp32part.py file
step 5 : run this cmd --> python  gen_esp32part.py <.CSV FILE> <OUTPUT BINARY FILE NAME >
                          eg : & C:/Python311/python.exe  .\gen_esp32part.py C:\Users\Admin\Documents\ArduinoData\packages\esp32\hardware\esp32\2.0.9\tools\partitions\default_ffat.csv D:\software\esp32_flasher_uti\flash_download_tool_3.9.5\flash_download_tool_3.9.5\bin
FLASHING_ADDRESS	--> 0x8000
Note : Here I am using FAT filesystem so i am converting default_ffat.csv to bin if you using littlefs or spiffs convert the proper .csv file
	   For SPIFFS use default.csv to convert it to .bin

file_name  			--> firmware.bin
file_loc   			--> your firmware loc
For creating .bin file click [sketch->exportcompiled binary] in the arduino IDE
FLASHING_ADDRESS	--> 0x10000


file_name           --> filesystem.spiffs.bin (or) filesystem.fatfs.bin
file_loc            --> C:\Users\Admin\AppData\Local\Temp\arduino_build_68202/filesystem.fatfs.bin
						eg : C:\Users\Admin\AppData\Local\Temp\arduino_build_68202/ESP32_iotmodgw_with_utility_ph2.fatfs.bin
FLASHING_ADDRESS for SPIFFS --> 0x290000 
FLASHING_ADDRESS for FATFS --> 0x291000 
Note : For file system we are not creating any binary file instead we loading the file from arduino at that time it will create .bin file in the above mentioned location , then we will use that .bin for flashing.
       >>>>> BY DEFAULT WE ARE USING SPIFFS FILE SYSTEM <<<<<
       
************************************ CREATING BIN FILES END ************************************