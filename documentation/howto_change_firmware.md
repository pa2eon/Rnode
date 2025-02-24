# How to compile firmware for the E22 module with ESP32-S3

## Dependencies

Make sure 'git', 'make', 'python' and 'pip' are instaled. Most like;y these are already availble on your system. Otherwise do:

    $ sudo apt install git
    $ sudo apt install make
    $ sudo apt install pip
    $ sudo apt install python

Download arduino-cli

## Define new board, the theory

In order to build custom firmware for a new board it has to be defined in several files.

### Makefile

BOARD_MODEL defines the target board. It is a unique 8 bit number.
BOARD_VARIANT defines the variant of the board. A board could come with different LoRa transceivers, for example. It is also a unique 8 bit number.

Every target board has a section 'firmware-<board_name>', 'upload-<board-name>' and 'release-<board-name>'
