mkdir /var/log/lighttpd
lighttpd -f /etc/lighttpd/lighttpd.conf

chmod +x /srv/www/htdocs/cgi-bin/*
rm -r /srv/www/htdocs/config/
ln -s /usrdata/cms/config /srv/www/htdocs/config
ln -s /usrdata/cms/log /srv/www/htdocs/log
ln -s /tmp /srv/www/htdocs/tmp

echo 0 > /usrdata/cms/config/web_status_flag

To set a gatway for our webpage through wifi
route delete 0.0.0.0 mask 0.0.0.0 192.168.10.100
route print
route -p add 192.168.10.100 mask 255.255.255.255 192.168.225.56 metric 11
ping 192.168.225.56
 
#should add in production version
mkdir -p /usrdata/cms/config
mkdir -p /usrdata/cms/cert
mkdir -p /usrdata/cms/bin
mkdir -p /usrdata/cms/log
echo "CMS_IOT_GW_12345" > /usrdata/cms/config/serial.txt

#for transferring the file form pc to unit manually
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\firmware\etc\banner" /etc/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\firmware\etc\init.d\rcS" /etc/init.d/rcS
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\firmware\home\root\startup.sh" /home/root/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\srv" /
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\IOT_GW_FW" /usrdata/cms/bin
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\schripts" /usrdata/cms/bin
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\certs\priya_mqtt.pem" /usrdata/cms/cert/SSL_CA_FILE.pem
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\bit_str_map_cfg.json" /usrdata/cms/config/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\val_to_str_map.json" /usrdata/cms/config/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\formula_cfg.json" /usrdata/cms/config/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\IOTGW_default_cfg.json" /usrdata/cms/config/IOTGW_default_cfg.json
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\IOTGW_main_cfg_priya_mqtt.json" /usrdata/cms/config/IOTGW_main_cfg.json
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\cfg\iotgw_tls_cfg.json" /usrdata/cms/config/iotgw_tls_cfg.json
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\lib" /usrdata/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\i2c-tools" /usrdata/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\bash_bin\bash" /bin/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\etc\lighttpd" /etc/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\IOTGW_QUECTEL\Lighttpd_web_page\json-c_lib\libjson-c.so.5" /lib/
.\adb.exe push "C:\Users\Admin\OneDrive - Creative Micro Systems\Sanjay_src\Projects\Quectel_EC25-E_with_FC20-Q73-RnD\wifi_example_exe" /usrdata/

#should add in production version
chmod +x  /etc/init.d/rcS
chmod +x /srv/www/htdocs/cgi-bin/*
chmod +x /home/root/*
mv  /usrdata/cms/bin/schripts/* /usrdata/cms/bin
rm -r /usrdata/cms/bin/schripts
chmod +x /usrdata/cms/bin/*
mv  /usrdata/lib/* /lib
rm -r /usrdata/lib/
chmod +x /lib/*
mv  /usrdata/i2c-tools/bin/* /bin
mv  /usrdata/i2c-tools/lib/* /lib
rm -r /usrdata/i2c-tools
chmod +x /bin/*
chmod +x /lib/*
echo 1 > /usrdata/cms/config/debuglog.cfg
echo "sample_serial" > /usrdata/cms/config/serial.txt

#!/bin/sh

# Get the PID of the hostapd process
hostapd_pid=$(pgrep hostapd)

if [ -z "$hostapd_pid" ]; then
    echo "hostapd is not running."
    exit 1
fi

# Check if multiple PIDs were returned and take the first one
hostapd_pid=$(echo "$hostapd_pid" | awk 'NR==1')

# Get the start time of the hostapd process in jiffies
start_time=$(awk '{print $22}' /proc/$hostapd_pid/stat 2>/dev/null)

if [ -z "$start_time" ]; then
    echo "Failed to get start time for hostapd (PID: $hostapd_pid)."
    exit 1
fi

# Get the current time in seconds since epoch
current_time=$(date +%s)

# Calculate elapsed time in seconds
elapsed_time=$((current_time - $(stat -c %Y /proc/$hostapd_pid)))

# Convert elapsed time into days, hours, minutes
days=$((elapsed_time / 86400))
hours=$(( (elapsed_time % 86400) / 3600 ))
minutes=$(( (elapsed_time % 3600) / 60 ))

# Get hostapd connection status
hostapd_status=$(hostapd_cli status | grep "state=")
connected_clients=$(hostapd_cli status | grep "num_sta[0]=")

# Display the information
echo "Uptime: ${days} days, ${hours} hours, and ${minutes} minutes"
echo "Connection Status: ${hostapd_status#state=}"
echo "Connected Clients: ${connected_clients#num_sta[0]=}"

