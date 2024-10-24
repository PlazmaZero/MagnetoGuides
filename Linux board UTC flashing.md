# Updating Linux/can Board
## Fixing the bootloader with a STLink
1. Turn off the printer
2. Connect the STLink to the canboard with the RST, SWCLK, SWDIO, GND, 3V3 pins.

   ![image](https://github.com/user-attachments/assets/1f58f566-6def-4c90-b827-b453092dc492)

4. Connect your STLink device to your USB port and open up STM32CubeProgrammer. Set the interface to ST-Link, setup the configuration like below, and hit connect. If you’ve never connected this STLink before, reset mode is the only option that is changed from default.

   ![image](https://github.com/user-attachments/assets/c60b00f5-dc76-4c4f-8b78-3704c114cb6d)

5. Your board should be connected now. Your window should look something like this (don’t worry about the exact values within Device Memory)

   ![image](https://github.com/user-attachments/assets/c12e491d-dc5f-4cf3-82d6-40b39e779899)

6. Now you need to click on the “OB” button on the left. Expand the User Configuration setting and scroll down until you find the nBOOT_SEL option. You will need to uncheck that, and then hit Apply

   ![image](https://github.com/user-attachments/assets/539f569b-fd50-4c8a-9a04-92f2cc4c1d0d)

7. If all goes well, you should get a message the option bytes were successfully programmed. Now your boot button should work as expected.

From here you can flash whatever you want to the device.


## Flashing Katapult to Linux/can board: 
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
`make clean KCONFIG_CONFIG=config.STM32G0B1`
3. Open menuconfig
`make menuconfig KCONFIG_CONFIG=config.STM32G0B1`

   ![image](https://github.com/user-attachments/assets/ea79e188-8d16-40bf-9363-9d0fa1086621)

5. Quit and save the configuration
6. Run the make command to compile the firmware
`make KCONFIG_CONFIG=config.STM32G0B1 -j4`
7. You should now have a katapult.uf2 file at ~/katapult/out/

### Flash Katapult firmware
1. Connect the linux hub USB C port to the pi while holding down the boot button

2. To confirm it’s in DFU mode you can run the command lsusb and look for an entry of “STMicroelectronics STM Device in DFU mode”

   ![image](https://github.com/user-attachments/assets/4cc16aa0-5f8f-4f65-b938-7eb0a0963a48)

3. You can then flash the Katapult firmware to your mainboard by running

`cd ~/katapult
make
sudo dfu-util -R -a 0 -s 0x08000000:leave -D ~/katapult/out/katapult.bin -d 0483:df11`

4. If the result shows an “Error during download get_status” or something, but above it it still has “File downloaded successfully” then it still flashed OK and you can ignore that error.

   ![image](https://github.com/user-attachments/assets/a2f18ba9-7d60-4f71-ba96-69383dd8a4f2)



## Flashing Klipper to the Linux/can board
### Compile Klipper for the Linux/can board
1. ssh into your klipper host console
2. cd to the Klipper directory
`cd ~/klipper`
4. Run make clean
`make clean KCONFIG_CONFIG=config.STM32G0B1`
5. Open menuconfig
`make menuconfig KCONFIG_CONFIG=config.STM32G0B1`

   ![image](https://github.com/user-attachments/assets/8475819d-1f1d-4eee-95e2-8c469f9bc967)

7. Quit and save the configuration
8. Run the make command to compile the firmware
`make KCONFIG_CONFIG=config.STM32G0B1 -j4`
6. You should now have a `klipper.bin` file at `~/home/pi/klipper/out`


### Flash the Klipper firmware to the toolhead board
1) Make sure the `klipper` service stopped.
`sudo service klipper stop`
3) Run an ls /dev/serial/by-id/ and take note of the Katapult device that it shows:
   ![image](https://github.com/user-attachments/assets/2f5e8d10-dbbf-4e5a-b8b9-51ab143a6940)

4) If the above command didn’t show a ‘katapult’ device, or threw a “no such file or directory” error, then quickly double-click the RESET button on your mainboard and run the command again. Until you get a result from a `ls /dev/serial/by-id/` there is no point doing further steps below.
5) Run this command to install klipper firmware via Katapult via USB. Use the device ID you just retrieved in the above ls command.
`python3 ~/katapult/scripts/flashtool.py -f ~/klipper/out/klipper.bin -d /dev/serial/by-id/usb-katapult_your_board_id`


 