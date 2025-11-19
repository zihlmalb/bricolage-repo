# Create AT-Binary for ESP-01S module with 1MB memory

I wasted some time trying to flash images onto an ESP-01 module to use it as a Wi-Fi interface. I tried some binaries found on the web, but none of them worked in the end. Finally, I tried to compile the esp-at project, which was successful.

![ESP-01S Module](../../../pics/esp01s.png 'ESP01-S Module')

## Prebuilt Binary images

If you don't want to build it yourself, just take the [binary images](https://github.com/zihlmalb/bricolage-repo/blob/main/data/esp-01s-1mb.tar.gz) and flash them using the esptool.

```sh
# extract the binary folder
tar -xzf ~/Downloads/esp-01s-1mb.tar.gz
cd esp-01s-1mb

# if esptool is already installed in your environment, skip
# the following 3 steps
python3 -m venv .venv
source .venv/bin/activate
pip install esptool

# flash the binaries
esptool -p /dev/ttyUSB1 -b 460800 --after hard_reset write_flash @flash_project_args

```
Make sure that you start the device in bootloader mode by pulling the GPIO0 pin to GND during power-up in order to flash the images.

If successful, go to [Testing](#testing).

## Build Instructions

Install the toolchain for xtensa-lx106 (for prerequisites, see <https://docs.espressif.com/projects/esp8266-rtos-sdk/en/latest/get-started/linux-setup.html>).
```sh
sudo apt-get install gcc git wget make libncurses-dev flex bison gperf python python-serial
mkdir ~/work
cd ~/work
wget https://dl.espressif.com/dl/xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz
tar -xzf xtensa-lx106-elf-gcc8_4_0-esp-2020r3-linux-amd64.tar.gz
echo export 'PATH="$PATH:$HOME/work/xtensa-lx106-elf/bin"' >> ~/.profile
```
Then clone the esp-at project and switch to the correct release tag:
```sh
 git clone https://github.com/espressif/esp-at.git
 cd esp-at
 git switch v2.3.0.0_esp8266 --detach
```
Then create and activate a virtual Python environment and install the required packages:
```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```
Afterwards, start build.py and choose the right platform ``PLATFORM_ESP8266`` and module ``WROOM-02-N``:
```sh
./build.py

Platform name:
1. PLATFORM_ESP32
2. PLATFORM_ESP8266
3. PLATFORM_ESP32S2
4. PLATFORM_ESP32C3

# choose 2 PLATFORM_ESP8266

Module name:
1. WROOM-02 (description: TX:15 RX:13)
2. WROOM-5V2L (description: 5V UART level)
3. ESP8266_1MB (description: No OTA)
4. WROOM-02-N (description: TX:1 RX:3)
5. WROOM-S2
6. ESP8266_QCLOUD (description: QCLOUD TX:15 RX:13)

# choose 4. WROOM-02-N (description: TX:1 RX:3)

Enable silence mode to remove some logs and reduce the firmware size?
0. No
1. Yes

# choose 0 or 1

```
build.py checks out esp-idf and recognizes that some Python packages are still missing. So install the missing packages:
```sh
pip install -r esp-idf/requirements.txt
```
Now copy the default configuration of the 1MB 8266 into the top-level directory:
```sh
cp module_config/module_esp8266_1mb/sdkconfig.defaults ./sdkconfig
```
Start build.py again, this time with the argument menuconfig:
```sh
./build.py menuconfig
```
Again, build.py interrupts and complains that ncurses is not installed. This is not the case since it was installed in one of the first steps. Just deactivate the check in the file esp-idf/tools/kconfig/lxdialog/check-lxdialog.sh: write `exit 0` instead of `exit 1` in line 75:
```sh
# Check if we can link to ncurses
check() {
        $cc -x c - -o $tmp 2>/dev/null <<'EOF'
#include CURSES_LOC
main() {}
EOF
	if [ $? != 0 ]; then
	    echo " *** Unable to find the ncurses libraries or the"       1>&2
	    echo " *** required header files."                            1>&2
	    echo " *** 'make menuconfig' requires the ncurses libraries." 1>&2
	    echo " *** "                                                  1>&2
	    echo " *** Install ncurses (ncurses-devel) and try again."    1>&2
	    echo " *** "                                                  1>&2
	    exit 0 # changed from exit 1 to exit 0
	fi
}

```
Then again:
```sh
./build.py menuconfig
```
If necessary, change the serial port for flashing:
Serial flasher config ---> Default serial port

Then leave Menuconfig and save the settings.
Build the project:
```sh
./build.py all
```
Note: There was a typo in the original command - corrected from `./buil.py all` to `./build.py all`.

## Flash

In order to flash the module, you have to pull GPIO0 to ground during power-up. This causes the controller to start in bootloader mode.

Then simply call the following command:

```sh
./build.py flash
```
You may have to change the serial port as described above with `build.py menuconfig`.

## Testing

If the flash command was successful, test the device. To do this, attach a serial terminal to the serial port where the module is plugged in:

* Baud rate: 115200
* Handshake: none
* Data bits: 8
* Stop bits: 1
* CR-LF at end of line

This can be achieved, for example, with picocom:
```sh
picocom --omap crcrlf --baud 115200 /dev/ttyUSB1
```
Repower the module, but now leave the GPIO0 pin open. You will then see the output ``ready``. You can then start to send your AT commands:

```at
AT

OK
```
Set the device to client mode and check the Wi-Fi networks in your environment:
```at
AT+CWMODE=1
AT+CWLAP
```
