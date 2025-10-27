# Updating MagnetoX Klipper version
## What to do after klipper update
1. install gcode shell command with KIAUH
2. **Update from Klipper 10 to Klipper 12 only**:
alter `klipper/klippy/extras/magneto_load_cell_py` line 16  from `self.load_cell_reset_pin.setup_start_value(1,1,False)` to `self.load_cell_reset_pin.setup_start_value(1,1)`
as the v12 klipper setup_start_value function only takes 2 arguements(vs three in v10).
3. Make sure magneto-service is set up and running. (Last section on page)

## Flashing Katapult to toolhead board: 
### Installing Katapult (also known as CANBoot)
I followed this guide from Mellow to get the Katapult repo, add to moonraker,  and install Katapult on the Lancer toolboard with the following settings: https://mellow-3d.github.io/fly_sb2040_v2_canboot_can.html
1. ssh into your klipper host console

2. Clone the Katapult firmware to your Klipper Host:
```
cd ~/
git clone https://github.com/Arksine/katapult
```
3. From your Fluidd or Mainsail web UI Edit Moonraker.conf and add the following at the bottom to allow Moonraker to manage updates to Katapult.
```
[update_manager Katapult]
type: git_repo
path: ~/katapult
origin: https://github.com/Arksine/katapult.git
is_system_service: False
```

### Compile Katapult firmware
1. cd to the Katapult directory
`cd ~/katapult`
2. Run make clean
`make clean KCONFIG_CONFIG=config.rp2040`
3. Open menuconfig
`make menuconfig KCONFIG_CONFIG=config.rp2040`

![image](https://github.com/user-attachments/assets/015dbbb5-a1e5-4d23-a699-6357ae133fc8)

5. Quit and save the configuration
6. Run the make command to compile the firmware
`make KCONFIG_CONFIG=config.rp2040 -j4`
7. You should now have a katapult.uf2 file at ~/katapult/out/

### Flash Katapult firmware
1. Disconnect all cables from the toolhead board and make sure the printer is off.
2. Prepare a USB C cable

> **Note** If you are using a MACOS system you may need to use a USB hub between the mac and the toolhead board for the drive to show up.

4. Press and hold the boot button on the toolboard.

![image](https://github.com/user-attachments/assets/ac6c48f9-7343-4722-b7c1-719ebff04718)

6. While holding, insert the USB C cable to the port
7. The LEDs on the board should light up and your computer should recognize an `RPI_RP2` drive.
![image](https://github.com/user-attachments/assets/1fee934b-d738-4b8e-8b15-527ea4281786)

8. Copy the downloaded .uf2/.bin firmware file to the RPI_RP2 disk. The drive will automatically eject after copying.

## Flashing Klipper to the toolhead board
### Compile Klipper for the toolhead board
1. ssh into your klipper host console
2. cd to the Klipper directory
`cd ~/klipper`
4. Run make clean
`make clean KCONFIG_CONFIG=config.rp2040`
5. Open menuconfig
`make menuconfig KCONFIG_CONFIG=config.rp2040`
<img width="567" alt="image" src="https://github.com/user-attachments/assets/441e90dd-7a5e-49d2-a0b6-c3206dc6024e" />


6. Quit and save the configuration
7. Run the make command to compile the firmware
`make KCONFIG_CONFIG=config.rp2040 -j4`
6. You should now have a `klipper.bin` file at `~/home/pi/klipper/out`


### Flash the Klipper firmware to the toolhead board
#### Option 1: USBC firmware install
1. Follow the instructions in ***Flash Katapult firmware*** section of the document to flash by loading the firmware over USB
#### Option 2: Use Katapult to load klipper into the toolhead board
1) Make sure the `klipper` service stopped.
`sudo service klipper stop`
3) Build Klipper with CAN support and with the a bootloader offset matching that
   of the "application offset" in Katapult. (Done in the previous section)
4) Enter the bootloader.  This will occur automatically if no program is detected.
   If you built Katapult with an alternative method of entry you may use that.
   If upgrading from a currently flashed version of Klipper the `flashtool.py`
   script will command the device to enter the bootloader (currently for CAN
   devices only).
3) Run the flash script:
   For CAN Devices:
   ```
   cd ~/katapult/scripts
   python3 flashtool.py -i can0 -f ~/klipper/out/klipper.bin -u <uuid>
   ```
   Replace <uuid> with the appropriate uuid for your can device. (Check magneto_device.cfg on your printer)
   If the device has not been previouisly flashed with Klipper, it is possible
   to query the bootloader for the UUID:

   ```
   flashtool.py -i can0 -q
   ```

   **NOTE: A query should only be performed when a single can node is on
   the network.  Attempting to query multiple nodes may result in transmission
   errors that can force a node into a "bus off" state.  When a node enters
   "bus off" it becomes unresponsive.  The node must be reset to recover.**

   For USB/UART devices:
   Before flashing, make sure pyserial is installed.  This step only needs to
   be performed once:
   ```
   pip3 install pyserial
   ```
   ```
   python3 flashtool.py -d <serial device> -b <baud_rate>
   ```
   Replace `<serial_device>` the the path to the USB/UART device you wish to
   flash.  The `<baud_rate>` is only necessary for UART devices, and defaults
   to 250000 baud if omitted.



## Flashing BTT Octopus Pro 1.1 STM32H723 Version
### Compile klipper for BTT Octopus Pro 1.1 STM32H723
1. ssh into your klipper host console
2. cd to the Klipper directory
`cd ~/klipper`
4. Run make clean
`make clean KCONFIG_CONFIG=config.STM32H723`
5. Open menuconfig
`make menuconfig KCONFIG_CONFIG=config.STM32H723`
![image](https://github.com/user-attachments/assets/cdc63619-7a28-436e-8a01-ebabe4590ce2)

6. Quit and save the configuration
7. Run the make command to compile the firmware
`make KCONFIG_CONFIG=config.STM32H723 -j4`
6. You should now have a `klipper.bin` file at `~/home/pi/klipper/out`

### Firmware Installation (Ripped from Voron Docs)

**Important**: Please write down these steps or bookmark this page - you might need to repeat the following steps if you update Klipper.

#### Option 1: SDcard Firmware Install

* Works regardless of USB vs UART
* Requires a microSD card

1. Execute these commands via SSH to rename the firmware file to `firmware.bin`:

   ```bash
   cd ~/klipper
   mv out/klipper.bin out/firmware.bin
   ```

   **Important:** If the file is not renamed, the firmware will not be updated properly. The bootloader looks for a file named `firmware.bin`.

2. Use a tool such as cyberduck or winscp to copy the firmware.bin file off your Pi, onto your computer.

   ![image](https://github.com/user-attachments/assets/e76a922e-e2da-4014-bb71-5796852aed1a)



3. Ensure that your sdcard is formatted FAT32  (NOT EXFAT!)
4. Copy **firmware.bin** onto the microSD card
5. Power off the Octopus
6. Insert the microSD card
7. Power on the Octopus
8. After a few seconds, the Octopus should be flashed
9. You can confirm that the flash was successful by running `ls /dev/serial/by-id`.  If the flash was successful, this should now show a klipper device, similar to:

   ![image](https://github.com/user-attachments/assets/885775a8-a60e-4912-bc32-d8d11885149c)

(note: this test is not applicable if the firmware was compiled for UART, rather than USB)

**Important:** If the Octopus is not powered with 12-24V, Klipper will be unable to communicate with the TMC drivers via UART and the Octopus will automatically shut down.

#### Option 2: DFU Firmware Install

* Requires a USB connection
* Requires the installation of an extra jumper on the Octopus
* Does NOT require an sdcard

1. Power off Octopus
2. Install the BOOT0 jumper
   
   <img width="567" alt="image" src="https://github.com/user-attachments/assets/9a991128-0b42-400a-a483-5f08c970c58b" />

4. Connect Octopus & Pi via USB-C
5. Power on Octopus
6. From your ssh session, run `cd ~/klipper` to make sure you are in the correct directory
7. Run `lsusb`. and find the ID of the dfu device. The device is typically named `STM Device in DFU mode`.
8. If you do not see a DFU device in the list, press the reset button next to the USB connector and run `lsusb` again.
9. Run ` sudo make KCONFIG_CONFIG=config.STM32H723 flash FLASH_DEVICE=1234:5678`, replacing 1234:5678 with the ID from the previous step. Note that the ID is in hexadecimal form; it only contains the numbers `0-9` and letters `A-F`.
10. Power off the Octopus
11. Remove the jumper from BOOT0 and 3.3V
12. Power on the Octopus
13. You can confirm that the flash was successful by running `ls /dev/serial/by-id`. If the flash was successful, this should now show a klipper device, similar to:

   ![image](https://github.com/user-attachments/assets/f4e824df-25cb-4cb4-b5d5-2caf1608a8fb)

   (note: this test is not applicable if the firmware was compiled for UART, rather than USB)



Then I compiled the klipper binary with these settings, and sent it over can with the instructions from the Katapult github: https://github.com/Arksine/katapult#uploading-klipper

#### Option 3: Katapult Install

1.
    ```
   cd ~/katapult
   make clean KCONFIG_CONFIG=config.STM32H723
   make menuconfig KCONFIG_CONFIG=config.STM32H723
    ```

2. 
   Compile the firmware with `make KCONFIG_CONFIG=config.STM32H723 -j4`. You will now have a katapult.bin at in your ~/katapult/out/katapult.bin.
   
   To flash, connect your board to the Pi via USB then put the board into DFU mode (this should be included in the hardware_config section for your board, but if it’s not then the user manual from the manufacturer should have the instructions).
   
   To confirm it’s in DFU mode you can run the command `lsusb` and look for an entry of “STMicroelectronics STM Device in DFU mode”
   
   <img width="705" height="41" alt="image" src="https://github.com/user-attachments/assets/624dc78a-66fd-41aa-ae32-9b5996b447d9" />


   You can then flash the Katapult firmware to your mainboard by running
   
   
    ```
    cd ~/katapult
   make
   sudo dfu-util -R -a 0 -s 0x08000000:mass-erase:force:leave -D ~/katapult/out/katapult.bin -d 0483:df11
   ```

    If the result shows an “Error during download get_status” or something, but above it it still has “File downloaded successfully” then it still flashed OK and you can ignore that error.

    If you get a different error and do not see a “File downloaded successfully” then try:

    `sudo dfu-util -R -a 0 -s 0x08000000:leave -D ~/katapult/out/katapult.bin -d 0483:df11`

<img width="700" height="111" alt="image" src="https://github.com/user-attachments/assets/28982ac5-d8b4-49f2-9228-c64f421d9826" />


Katapult should now be successfully flashed. Take your mainboard out of DFU mode (it might require removing jumpers and rebooting, or just rebooting). Check that Katapult is installed and by running

`ls /dev/serial/by-id/*`

<img width="622" height="33" alt="image" src="https://github.com/user-attachments/assets/61206fb7-9f95-44e7-a344-6594d5e5aaa8" />


You should see a “usb-katapult_…” device there. If you don’t, then double-click the RESET button on your board and ls /dev/serial/by-id again.

 ##### Katapult Config
<img width="816" height="270" alt="image" src="https://github.com/user-attachments/assets/2cf74912-d2bf-4958-8bcd-3edad3603268" />


##### Klipper flashing

Stop the Klipper service on the Pi by running:

`sudo service klipper stop`

Move into the klipper directory on the Pi by running:

`cd ~/klipper`

Then go into the klipper configuration menu by running:

```
make clean KCONFIG_CONFIG=config.STM32H723
make menuconfig KCONFIG_CONFIG=config.STM32H723
```

##### Klipper Config
<img width="656" height="162" alt="image" src="https://github.com/user-attachments/assets/bda11ba2-4c12-4ebe-903d-86d00e51f0e9" />

You can now flash Klipper to your board using the Katpult /dev/serial ID you found earlier by running:

`make KCONFIG_CONFIG=config.STM32H723 flash FLASH_DEVICE=/dev/serial/by-id/usb-katapult_your_board_id`

<img width="1018" height="53" alt="image" src="https://github.com/user-attachments/assets/61e36b44-ac4a-4b5b-9166-8a71aabc60a2" />


<img width="571" height="296" alt="image" src="https://github.com/user-attachments/assets/4a91d8a6-0db3-4854-b321-ca6eb23ee885" />


Don’t worry about the “CanBoot” or “CAN Flash Success”, we aren’t flashing anything CANBUS, this is just an idiosyncracy of the klipper make flash tool.

Klipper should now be successfully flashed. Check that it is installed and by running

`ls /dev/serial/by-id/*`

<img width="586" height="38" alt="image" src="https://github.com/user-attachments/assets/9f23feef-01ef-45ef-9dc2-2d7db07e244f" />


You should see a “usb-klipper_…” device there instead of “usb-katapult_…”

Use this USB ID in the [mcu] section of your printer.cfg in order for Klipper (on Pi) to connect to the mainboard. (If this is a secondary board, like a second SKR for Z motors, or a Toolhead board, then make sure to name the mcu section, [mcu some_name] as per the Klipper docs).

<img width="720" height="84" alt="image" src="https://github.com/user-attachments/assets/4a8f0829-69cf-4222-a360-e20dfa92157e" />


Start the Klipper service on the Pi again by running:

`sudo service klipper start`

## Updating octopus via katapult
Stop the Klipper service on the Pi by running:

`sudo service klipper stop`

Move into the klipper directory on the Pi by running:

`cd ~/klipper`

Then go into the klipper configuration menu by running:

```
make clean KCONFIG_CONFIG=config.STM32H723
make menuconfig KCONFIG_CONFIG=config.STM32H723
```

You can find screenshots of settings for common toolheads in the Hardware Config section.

Otherwise, check the user manual for your board as it should list the proper klipper menuconfig settings.

Once you have the correct options selected, press Q to quit the menu (it will ask to save, choose yes).

You can now flash Klipper to your board using the existing Klipper /dev/serial ID you have in your printer.cfg file:

<img width="720" height="84" alt="image" src="https://github.com/user-attachments/assets/aaa7dd31-9f06-43d8-b5f4-73df2eab3016" />


or you can find it by running `ls /dev/serial/by-id/*`

<img width="586" height="38" alt="image" src="https://github.com/user-attachments/assets/e23bcbde-8db0-4cb0-96cf-ec45581c3bfb" />


Then you can simply flash klipper using this device ID by running:

`make KCONFIG_CONFIG=config.STM32H723 flash FLASH_DEVICE=/dev/serial/by-id/usb-Klipper_your_board_id`

<img width="997" height="56" alt="image" src="https://github.com/user-attachments/assets/fd938bfc-4764-403c-8071-7639b3bc93f2" />


<img width="571" height="296" alt="image" src="https://github.com/user-attachments/assets/1f925ba2-19cd-4dc7-a3a1-bf1a0d3e7a02" />


Don’t worry about the “CanBoot” or “CAN Flash Success”, we aren’t flashing anything CANBUS, this is just an idiosyncracy of the klipper make flash tool.

Klipper should now be successfully flashed. Double check that it still shows up as a Klipper device:

`ls /dev/serial/by-id/*`

<img width="586" height="38" alt="image" src="https://github.com/user-attachments/assets/5eff1993-8b14-41ab-8829-183faca5afea" />


Then start the Klipper service on the Pi again by running:

`sudo service klipper start`



## Create the magneto service file
### Prerequisite files from orginal magneto(not needed if updating with stock magneto hardware)
```
/home/pi/klipper/klippy/extras/gcode_shell_command.py /home/pi/klipper/klippy/extras
/home/pi/klipper/klippy/extras/magneto_load_cell.py /home/pi/klipper/klippy/extras
/home/pi/auto-uuid /home/pi
/home/pi/printer_data/config /home/pi/printer_data
```
### Making sure magneto service file is present and active
`sudo nano /etc/systemd/system/magneto.service`
1. Copy and paste the text below into the nano editor
```
[Unit]
Description=Magneto Manager Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/bin/bash /home/pi/auto-uuid/magneto-run.sh

[Install]
WantedBy=multi-user.target
```
2. Install Py serial and Flask for the service (Installed to magneto by default)
`sudo pip install flask`
`sudo pip install pyserial`
3. Change file permissions
Now, we need to change the file permissions to make it readable by all by typing
`sudo chmod 644 /etc/systemd/system/magneto.service`
4. As the last step, you need to tell the system that you have added this file and want to enable this service so that it starts at boot.
```
sudo systemctl daemon-reload
sudo systemctl enable magneto.service
systemctl status magneto
```
5. The service should now be running


 
