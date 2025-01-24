# Pico2-input_shaper
 Setup of a RPI Pico 2 to use as an input shaper for 3D printers
 
 Pico 2 Image Shaper Build Notes

Using standard cat6 cable with the Blue/Blue Stripe wires removed (not needed only require 6 wires)

Cable Color = Orange (Black wire on picture below)
ADXL345  Pin = GRD
Pico Pin# = 38

Cable Color = Orange Stripe (Red wire on picture below)
ADXL345  Pin = VCC
Pico Pin# = 36

Cable Color = Green (Green wire on picture below)
ADXL345  Pin = SDO
Pico Pin# = 1

Cable Color = Green Stripe (Blue wire on picture below)
ADXL345  Pin = CS
Pico Pin# = 2

Cable Color = Brown (Yellow wire on picture below)
ADXL345  Pin = SDA
Pico Pin# = 5

Cable Color = Brown Stripe (Orange wire on picture below)
ADXL345  Pin = SCL
Pico Pin# = 4
￼

To flash on Pico 2. Put the Pico 2 info DFU mode connecting it to RPI.
SSH into the RPI with Klipper installed and run commands below.
NOTE: The pico 2 will not work with dropping the UF2 file info the drive like normal so you must flash it with the final command below.

cd ~/klipper
make clean
make menuconfig
	- Do not enable extra low level options
	- Raspberry Pi 2040/2350
	- rp2350 processor model
	- no bootloader
	- usbserial
save file and exit
make flash FLASH_DEVICE=2e8a:000f

Once the flash is done disconnect the Pico 2 from the RPI and reconnect (not in DFU mode).

Run the command below to get the MCU ID info for the adxl.cfg file created in the printers mainsail interface.

ls /dev/serial/by-id/*

It will be something like…

/dev/serial/by-id/usb-klipper_rp2350_EE6837A84779EDB221-if00

Note this MCU ID for the adxl.cfg file.
Create the adxl.cfg file below in the machine config screen of your mainsail interface of the printer.

##
[mcu adxl]
serial: /dev/serial/by-id/ PUT MCU ID HERE!!
[adxl345]
cs_pin: adxl:gpio1
spi_bus: spi0a
axes_map: x,z,y

[resonance_tester]
accel_chip: adxl345
probe_points:
    150,150, 20  # middle of bed as an example
##

Save the new adxl.cfg file (do not reboot klipper yet). 
Open the printer.cfg file and add the command below anywhere in the file. Then save and restart klipper.
*** IF YOU DISCONNECT THE ADXL345 FROM THE PRINTER YOU MUST COMMENT OUT THE LINE BELOW IN THE PRINTER.CFG FILE WITH A #  BEFORE REMOVING THE CONNECTION  OR KLIPPER WILL CRASH ***

[include adxl.cfg]

Go to the console for the printer and type the command below to see if everything is communicating right.

ACCELEROMETER_QUERY


NOTE: 
In order to use input sharper with klipper on a RPI you need to run the following commands to install the required software on the RPI.

sudo apt update
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev

Then install NumPi with the following command.

~/klippy-env/bin/pip install -v "numpy<1.26"

Measuring the resonances by entering the command below into the console of the printer. 
First mount the ADXL345 to the X axis somewhere (double sided tape works well).

TEST_RESONANCES AXIS=X

Once the test is done move the ADXL to the Y axis somewhere. Then run the command below.

TEST_RESONANCES AXIS=Y

Run the below scripts on the RPI to create the averaging results PNG image files. 

~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o /tmp/shaper_calibrate_x.png
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_y_*.csv -o /tmp/shaper_calibrate_y.png

This will create two PNG image charts (one for X axis and one for Y axis) with the results of the resonances tests. They will be saved in the following location on the RPI.

/tmp/shaper_calibrate_x.png
/tmp/shaper_calibrate_y.png

Use the information in the charts to set the input_shaper settings in the printer.cfg file.

[input_shaper]
shaper_freq_x: 
shaper_type_x: 
shaper_freq_y: 
shaper_type_y: 


Pico board reference = https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html

