# Pico2-input_shaper
Setup of a RPI Pico 2 to use as an input shaper for 3D printers<br>
Pico 2 Image Shaper Build Notes. This project is for the Pico 2 only. This Git was created because there is no good documentation in one place to get this build done for the Pico 2. Some of the commands are slightly different than doing it with a regular Pico. Hope it helps.<br>
<br>
For the cable just use a standard cat6 network cable with the Blue/Blue Stripe wires removed (not needed only require 6 wires)<br>
<br>
Cable Color = Orange (Black wire on picture below)<br>
ADXL345  Pin = GRD<br>
Pico Pin# = 38<br>
<br>
Cable Color = Orange Stripe (Red wire on picture below)<br>
ADXL345  Pin = VCC<br>
Pico Pin# = 36<br>
<br>
Cable Color = Green (Green wire on picture below)<br>
ADXL345  Pin = SDO<br>
Pico Pin# = 1<br>
<br>
Cable Color = Green Stripe (Blue wire on picture below)<br>
ADXL345  Pin = CS<br>
Pico Pin# = 2<br>
<br>
Cable Color = Brown (Yellow wire on picture below)<br>
ADXL345  Pin = SDA<br>
Pico Pin# = 5<br>
<br>
Cable Color = Brown Stripe (Orange wire on picture below)<br>
ADXL345  Pin = SCL<br>
Pico Pin# = 4<br>
<br>
  <img align="center" src="https://github.com/Scott1974-1/Pico2-input_shaper/blob/main/images/adxl345-pico.jpg" /> <br>
<br>
To flash on Pico 2. Put the Pico 2 info DFU mode connecting it to RPI.<br>
SSH into the RPI with Klipper installed and run commands below.<br>
NOTE: The pico 2 will not work with dropping the UF2 file info the drive like normal so you must flash it with the final command below.<br>
<br>
cd ~/klipper<br>
make clean<br>
make menuconfig<br>
<br>
<img align="center" src="https://github.com/Scott1974-1/Pico2-input_shaper/blob/main/images/klipper_pico_menuconfig.png" /> <br>
<br>
	- Do not enable extra low level options<br>
	- Raspberry Pi 2040/2350<br>
	- rp2350 processor model<br>
	- no bootloader<br>
	- usbserial<br>
save file and exit<br>
make flash FLASH_DEVICE=2e8a:000f<br>
<br>
Once the flash is done disconnect the Pico 2 from the RPI and reconnect (not in DFU mode).<br>
Run the command below to get the MCU ID info for the adxl.cfg file created in the printers mainsail interface.<br>
<br>
ls /dev/serial/by-id/*<br>
<br>
It will be something like…<br>
<br>
/dev/serial/by-id/usb-klipper_rp2350_EE6837A84779EDB221-if00<br>
<br>
Note this MCU ID for the adxl.cfg file.<br>
Create the adxl.cfg file below in the machine config screen of your mainsail interface of the printer.<br>
<br>
##<br>
[mcu adxl]<br>
serial: /dev/serial/by-id/ PUT MCU ID HERE!!<br>
[adxl345]<br>
cs_pin: adxl:gpio1<br>
spi_bus: spi0a<br>
axes_map: x,z,y<br>
<br>
[resonance_tester]<br>
accel_chip: adxl345<br>
probe_points:<br>
    150,150, 20  # middle of bed as an example<br>
##<br>
<br>
Save the new adxl.cfg file (do not reboot klipper yet). <br>
Open the printer.cfg file and add the command below anywhere in the file. Then save and restart klipper.<br>
<b> *** IF YOU DISCONNECT THE ADXL345 FROM THE PRINTER YOU MUST COMMENT OUT THE LINE BELOW IN THE PRINTER.CFG FILE WITH A #  BEFORE REMOVING THE CONNECTION  OR KLIPPER WILL CRASH *** </b><br>
<br>
[include adxl.cfg]<br>
<br>
Go to the console for the printer and type the command below to see if everything is communicating right.<br>
<br>
ACCELEROMETER_QUERY<br>
<br>
<br>
NOTE: <br>
In order to use input sharper with klipper on a RPI you need to run the following commands to install the required software on the RPI.<br>
<br>
sudo apt update<br>
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev libopenblas-dev<br>
<br>
Then install NumPi with the following command.<br>
<br>
~/klippy-env/bin/pip install -v "numpy<1.26"<br>
<br>
Measuring the resonances by entering the command below into the console of the printer. <br>
First mount the ADXL345 to the X axis somewhere (double sided tape works well).<br>
<br>
TEST_RESONANCES AXIS=X<br>
<br>
Once the test is done move the ADXL to the Y axis somewhere. Then run the command below.<br>
<br>
TEST_RESONANCES AXIS=Y<br>
<br>
Run the below scripts on the RPI to create the averaging results PNG image files. <br>
<br>
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o /tmp/shaper_calibrate_x.png<br>
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_y_*.csv -o /tmp/shaper_calibrate_y.png<br>
<br>
This will create two PNG image charts (one for X axis and one for Y axis) with the results of the resonances tests. They will be saved in the following location on the RPI.<br>
<br>
/tmp/shaper_calibrate_x.png<br>
/tmp/shaper_calibrate_y.png<br>
<br>
EXAMPLE:<br>
<img align="center" src="https://github.com/Scott1974-1/Pico2-input_shaper/blob/main/images/calibrate-y.png" /> <br>
<br>
Use the information in the charts to set the input_shaper settings in the printer.cfg file.<br>
<br>
[input_shaper]<br>
shaper_freq_x: <br>
shaper_type_x: <br>
shaper_freq_y: <br>
shaper_type_y: <br>
<br>
<br>
Pico boards reference: https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html<br>
Klipper measuring resonances docs: https://www.klipper3d.org/Measuring_Resonances.html<br>
<br>
See the STL files directory in this github for the files of the cases used in this project.<br>
<br>
<br>
Below are a few images of the final product. very clean and sleek design. Also very durable.<br>
This can be used on multiple printers by just creating the adxl.cfg file on all the printers and adding the line to the printer.cfg file.<br>
<b>*** Just make sure you comment out the line in the printer.cfg file BEFORE disconnecting the USB cable or klipper will crash *** </b><br>
<br>
<img align="center" src="https://github.com/Scott1974-1/Pico2-input_shaper/blob/main/images/IMG_0635.JPG" /> <br>
<br>
</br>
<img align="center" src="https://github.com/Scott1974-1/Pico2-input_shaper/blob/main/images/IMG_0636.JPG" /><br>
<br>


