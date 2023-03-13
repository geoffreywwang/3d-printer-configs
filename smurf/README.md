# Smurf

Steps to setup a new Raspberry Pi for Smurf:


### Creating rpi image
- Download `Raspberry Pi Imager`
- Select `Raspberry Pi OS Lite` from the `Raspberry Pi OS (other)` menu
	- Probably should use `Raspberry Pi OS Lite (64-bit)` but I didn't realize that was an option and installed `Raspberry Pi OS Lite (32-bit)`
- Click the gear icon on the bottom right:
	- Set hostname to `smurf`
	- Enable SSH (Use password authentication)
		- Could be changed to public-key if more security is desired
	- Set username and password
	- Configure wireless
	- Set locale
- Select SD card and flash


### General rpi software
- Put SD card in rpi
- Either connect mouse, keyboard, and HDMI or use SSH
- Connect rpi to the network (WIFI or Ethernet)
- Run the following commands
	- `sudo apt update`
	- `sudo apt upgrade`
	- `sudo apt install git neovim`
- Then run `sudo raspi-config`
	- Add wifi network
	- Set keyboard in localization options to use US and not UK layout
- Reboot rpi with `sudo reboot now`


### Installing Klipper and extra software
- Download and install [KIAUH](https://github.com/th33xitus/kiauh)
	- Install Klipper
	- Moonraker
	- Fluidd
- Install [Mobileraker Companion](https://github.com/Clon1998/mobileraker_companion)


### Flashing MCU
Information pulled from Voron [website](https://docs.vorondesign.com/build/software/miniE3_v20_klipper.html). Smurf uses a `BTT SKR mini E3 V2.0`

- Run the following:
```
cd ~/klipper
make menuconfig
```
- In the menu structure there are a number of items to be selected.
	- Ensure that the micro-controller architecture is set to ‘STM32’
	- Ensure that the processor model is set to ‘STM32F103’
	- Ensure that the Bootloader offset is set to ‘28KiB’
	- Ensure that the Clock Reference is set to ‘8 Mhz’
	- Ensure that “Enable extra low-level configuration options” is selected.
	- Ensure that “Use USB for communication (instead of serial)” is selected.
	- Ensure that “GPIO pins to set at micro-controller startup” includes ‘!PA14’.

- Then run:
```
make clean
make
mv klipper.bin ~/printer_data/config
```

- Open up fluidd dashboard (`http://smurf.local` or `http://{IP_GOES_HERE}`)
- Go to configurations page and download `klipper.bin`
- Rename `klipper.bin` to `firmware.bin`
- Power off Smurf
- Take out SD card from MCU board
- Put `firmware.bin` on SD card
- Insert SD card into MCU board and power up Smurf
- Smurf should automatically install Klipper firmware onto the MCU


### Flashing Raspberry Pi (to support accelerometer for input shaping)
Information pulled from Klipper [website](https://www.klipper3d.org/RPi_microcontroller.html).

- Run the following:
```
cd ~/klipper/
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service

make menuconfig
```
- Set "Microcontroller Architecture" to "Linux process" and remove all other options
- Then run:
```
sudo service klipper stop
make flash
sudo service klipper start
```
- Run `sudo raspi-config` again and enable `SPI` under `Interfacing options`


### Add printer configs
Copy the following files into `~/printer_data/config`
- `fluidd.cfg`
- `neopixels.cfg`
- `printer.cfg`