# AP121-OpenWRT-Install
 How to Install OpenWRT on AeroHive AP121 Wireless Router.

 This is a short guide on how I installed and configured my 4 Aerohive AP121's with OpenWRT 22.03.3 in January 2023. This guide will work regardless if the AP's Serial Number is still bound to its original owners ExtremeCloud IQ account/network.

 Note: This guide is not for beginners, if you are unsure about what you are doing please read the whole guide before attempting.

 The usual Disclaimer: USE THIS GUIDE AT YOUR OWN RISK, I DO NOT ACCEPT ANY RESPONSIBILITY FOR BRICKED/BROKEN HARDWARE.

 ## Hardware Requirements;

- Aerohive AP121 - Working
- Cisco Console Cable (RJ45 - DB9 Serial - NOTE: As most PC's don't have RS232 DB9 connectors anymore, I bought an RJ45 - USB [via FTDI232RL+ZT213LEEA chips] Console cable.) [Console Cable](https://www.amazon.co.uk/dp/B07VMD9L3C?psc=1&ref=ppx_yo2ov_dt_b_product_details)
- 802.3af Gigabit POE adapter (for powering the AP) [TP-Link Example](https://www.amazon.co.uk/TL-PoE160S-Injector-Supplies-Wall-Mount-Distance/dp/B08LQP8CYD/ref=sr_1_3?keywords=gigabit+poe+injector&qid=1675436863&sprefix=gigabit+poe%2Caps%2C62&sr=8-3)
**OR**
- Centre Positive Barrel Jack 12v 2A DC Power Supply (The AP is rated for 1.1A, but I used 2A supply with no issues)
- RJ45 Network Cable
- RJ45 Network Port (Cannot be done over Wi-Fi, ie netbooks)
    
## Software Requirements;

- PuTTY Serial Console (This is included in the standard version of PuTTY SSH Console)
- TFTPD64 TFTP Server
- Static IP (Details Below)
- OpenWRT Image for AP121 - https://downloads.openwrt.org/releases/22.03.3/targets/ath79/nand/openwrt-22.03.3-ath79-nand-aerohive_hiveap-121-squashfs-sysupgrade.bin

## Initial Setup;

1: Download OpenWRT Image for Aerohive AP121.

2: Install PuTTY & TFTPD64

3: Disconnect all networks from Computer, go to network settings and set the network adapters IP Address to Static IP 192.168.1.10.

4: Open TFTPD64, go to settings and set "Bind TFTP to this Address" to 192.168.1.10 (the box is a dropdown menu).

5: Create a folder in documents called OpenWRT, place the OpenWRT Firmware image inside this folder. Then, in TFTPD64 set the "Current Directory" to the OpenWRT Folder in Documents.

6: Plug in your RJ45 - USB console cable to your PC's USB Port and identify the COM Port its using(for example mine was using COM6 - you can use device manager for this)

7: Plug one end of the RJ45 Network cable into the ETH0 on the AP121, and the other into your PC's Network Port.

8: Plug the RJ45 console cable into the CONSOLE port on the AP121.

9: Launch PuTTY and select Serial, input your COM port (ie COM6) and ensure the Baud rate is set to 9600. Press start and Click the blank window to ensure its highlighted.

10: Plug the DC Jack into the 12v plug on the AP121, but do not turn on yet.

## Software Setup;

1: When you are ready, Supply Power to the AP121 and watch the PuTTY serial console window for activity. Soon it should display some text and say "Press any key to stop boot". Press any key NOW.

2: You will be prompted for the console password, it is either administrator OR AhNf?d@ta06 (this was the one my AP worked with). You should now be logged in.
    
3: Run command 'Version'

- NOTE: The following bootloaders are listed as compatible with OpenWRT (Via their website), these are:
    v1.0.0.43 (Supplied with HiveOS 6.2r1)
    v1.0.0.4f (Supplied with HiveOS 6.5r4)
    v1.0.0.50 (Supplied with HiveOS 6.5r3)
    v1.0.0.52 (Supplied with HiveOS 6.5r8b) - This was the firmare my router used.
    IF THE FIRMARE IS LISTED LOWER THAN V1.0.0.43: STOP NOW, your bootloader probably isn't compatible.

4: Transfer the OpenWRT image to the AP's memory. Be Patient, it might take a while.
    tftpboot 0x81000000 openwrt-22.03.3-ath79-nand-aerohive_hiveap-121-squashfs-factory.bin" (Change to your file name)

NOTE: The next instructions erase/write NAND, this is permenant and if done wrong may brick the router's firmware or cause OpenWRT to not boot properly (although this is recoverable.)

5: Erase the NAND Flash in the Access Point to make way for OpenWRT using the command;
    nand erase 0x800000 0x7400000

- Command Breakdown: 0x800000 is the flash memory address in the Access Point.
                    0x7400000 is the filesize. This is important later.

6: Write the OpenWRT Image to NAND Flash in the AP using the command;
    nand write 0x81000000 0x800000 0x7400000

- Command Breakdown: 0x8100000 is the memory address that the OpenWRT image is stored in.
                     0x800000 is the flash memory address for the Access Point.
                     0x7400000 is the filesize.

NOTE: Some guides list the filesize as 0x800000 as well as the address, this is wrong and will result in a semi-corrupted OpenWRT install (will show a kernel panic during boot) as only the first 0x800000 was written too. If you have done this, rerun the erase/write command with 0x7400000.

7: If you have done 5: and 6: correctly and recieved "OK" after each, run the command;
    reset

- NOTE: During first reboot, if the AP was booting via its secondary backup flash chip located at 0xd00000 then the boot may fail with a Bad Magic number error. Do not panic, allow the boot process to fail 3 times, after which the AP will then default to booting from 0x800000 and should boot OpenWRT Normally.

8: During Boot watch for the orange light to turn Flashing white. This means the boot process is occuring, monitor the console for errors. If the LED turns solid white then the AP has successfully booted - this may take 90s or more.

9: You can now navigate to 192.168.1.1 in the browser and view the LUCi interface to configure the AP.
    

## OpenWRT (LUCi) Configuration:

1: OpenWRT web interface default password is:
    passwd
CHANGE IT!

2: Navigate to;
    Network > Wireless
- The Atheros AR9340 802.11bgn is the 2.4GHz Wireless Adapter.
- The Generic MAC80211 802.11an is the 5GHz Wireless Adapter.

3: Press Edit on either SSID, Change the operating frequency width to 40MHz. This will get you more speed (almost 2x), although if your having issues with congestion change this to 20MHz.

4: Change the ESSID to a Name of your choice, this is your network ID that all devices will see. I used;
    Aerohive Mesh

5: Navigate to Wireless Security, select WPA3-SAE and set a password in the "KEY" box.

6: Repeat 3: 4: & 5: for the other SSID (i have my 2.4 & 5GHz networks seperate, using the "Legacy" name to denote the 2.4GHz network - ie Aerohive Legacy.)

7: Enable both SSID's using the enable button.

8: On your Mobile, ensure the SSID you just setup shows up and can be connected to.

12: Navigate to;
    System > System

10: Under General Settings, change the hostname to whatever you like. This will be the name the AP presents to the network in order for it to be easier identified next to its IP address. As I have 4 Aerohives i names mine Aerohive1 - 4.

11: Save and Apply

12: Navigate to;
    Network > Interfaces

13: In interfaces, 1 LAN should be listed as "BR-LAN". Press Edit.

14: Change the Protocol to;
    DHCP-Client
This will disable the DHCP server on the access point and force it to get an IP address from your existing router

15: Navigate to firewall settings and under the Create / Assign firewall-zone dropdown, select;
    unspecified
This will disable the AP's local firewall - fine as your main Router handles it.

16: Press Save and Apply. This will bring up a menu warning about connectivity change. If you press;
    Apply with revert after connectivity loss
You will have 90s to unplug the router network cable from your PC, replug into your existing network, and get LUCi up on a device to accept the changes.

The other button is;
    Apply and keep settings
This will just apply the changes and not check the new link works. I use this as i restart the router before i plug into my existing network.

17: Unplug the AP, then plug into your existing network. I usually power off the AP and restart the AP during this.

18: Use Advanced IP Scanner (Windows) or LanScan (MacOS) to identify the AP's new IP address. it should show up as your set Hostname, with the adapter listed as "Extreme Networks, Inc"

19: Navigate back to the LUCi Interface and check settings.

20: Connect WiFi device to the AP SSID, go to Fast.com and run. 
- My Network speed from 1m away from the router using 5GHz is 200Mbps (with a 1Gbps WAN) on my iPhone 13. (If you get 100Mbps, check that your SSID operating frequency width is set to 40MHz, not 20MHz).
- My Network speed from 1m away from the router using 2.4GHz is 75Mbps (with a 1Gbps WAN) on my iPhone 13. (If you get 30Mbps, check that your SSID operating frequency width is set to 40MHz, not 20MHz).

###### You have now successfully setup your Aerohive AP121 with OpenWRT!







