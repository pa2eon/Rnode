# How to compile firmware for the E22 module with ESP32-S3

## Dependencies

Make sure 'git', 'make', 'python' and 'pip' are instaled. Most like;y these are already availble on your system. Otherwise do:

    $ sudo apt install git
    $ sudo apt install make
    $ sudo apt install pip
    $ sudo apt install python

Download arduino-cli

    $ cd
    $ curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

Add arduino-cli to path by editing ~/.bashrc

    $ nano ~/.bashrc
    add 'export PATH=~/bin:$PATH' to end of file
    $ test the working : #arduino-cli <r>

Install the esp32 extra 'tools'

    $ arduino-cli core install esp32:esp32

## Clone git repo (keep the name structure in the same format as the repo)

    $ git clone https://github.com/liberatedsystems/RNode_Firmware_CE.git

## Prepare

Install the required BSP and libraries for the ESP32 system.

    $ cd RNode_Firmware_CE/
    $ make prep-esp32

Add rns software to path:

    $ nano ~/.bashrc

    add 'export PATH=~/.local/bin:$PATH' to the end of the file.

## Test

To test if you can compile the firmware try it:

    $ make firmware-heltec32_v3

Note: The RX buffer value must be changed - */BluetoothSerial.cpp to 6144
      The TX buffer value must be changed - */BluetoothSerial.cpp to 384    

After this command you will find an compiled folder: $ ~/RNode_Firmware_CE/build/heltec32_v3

    $ make upload-heltec32_v3  - to bring the firmware to the hardware !

To check if the firmware is good installed on the hardware:

    $ nodeconf /dev/ttyACM0 -i




## Define new board, the theory

In order to build custom firmware for a new board it has to be defined in several files.

### Makefile

BOARD_MODEL defines the target board. It is a unique 8 bit number.
BOARD_VARIANT defines the variant of the board. A board could come with different LoRa transceivers, for example. It is also a unique 8 bit number.

Every target board has a section 'firmware-<board_name>', 'upload-<board-name>' and 'release-<board-name>'
