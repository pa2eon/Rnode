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
    $ test the working : # arduino-cli <r>

## Clone git repo

    $ git clone https://github.com/liberatedsystems/RNode_Firmware_CE.git

## Prepare

Install the required BSP and libraries for the ESP32 system.

    $ cd RNode_Firmware_CE/
    $ make prep-esp32

This command stalled. I stopped the command by hiting CTRL-C and restarted it.

Add rns software to path:

    $ nano ~/.bashrc

    add 'export PATH=~/.local/bin:$PATH' to the end of the file.

## Test

To test if you can compile the firmware try it:

    $ make firmware-heltec32_v3

Fingers crossed!

## Define new board, the theory

In order to build custom firmware for a new board it has to be defined in several files.

### Makefile

BOARD_MODEL defines the target board. It is a unique 8 bit number.
BOARD_VARIANT defines the variant of the board. A board could come with different LoRa transceivers, for example. It is also a unique 8 bit number.

Every target board has a section 'firmware-<board_name>', 'upload-<board-name>' and 'release-<board-name>'
