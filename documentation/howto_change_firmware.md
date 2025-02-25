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

Note: The RX buffer value must be changed (near line 43) - */libraries/BluetoothSerial/src/BluetoothSerial.cpp to 6144 <br>
      The TX buffer value must be changed (near line 44) - */libraries/BluetoothSerial/src/BluetoothSerial.cpp to 384 <br>   

After this command you will find an compiled folder: $ ~/RNode_Firmware_CE/build/heltec32_v3

    $ make upload-heltec32_v3  - to bring the firmware to the hardware !

To check if the firmware is good installed on the hardware:

    $ nodeconf /dev/ttyACM0 -i


## Define new board, the theory

In order to build custom firmware for a new board it has to be defined in several files.

### Makefile

BOARD_MODEL defines the target board. It is a unique 8 bit number.
BOARD_VARIANT defines the variant of the board. A board could come with different LoRa transceivers, for example. It is also a unique 8 bit number.

Every target board has a section 'firmware-<board_name>', 'upload-<board-name>' and 'release-<board-name>' <br>
Then the Arduino version in line 17 can be changed to ( for example from 2.0.17 to 3.11.x ) - check later !!


### boards.h

Both BOARD_MODEL and BOARD_VARIANT are #defines in this file. 
They are used to define the pinout of the SPI port and if it has certain perifirals such as a screen BLE etc.

Let's see an example:

    #elif BOARD_MODEL == BOARD_HELTEC32_V3
      #define IS_ESP32S3 true
      #define HAS_DISPLAY true
      #define DISPLAY OLED
      #define HAS_BLUETOOTH false
      #define HAS_BLE true
      #define HAS_PMU true
      #define HAS_CONSOLE true
      #define HAS_EEPROM true
      #define HAS_INPUT true
      #define HAS_SLEEP true
      #define PIN_WAKEUP GPIO_NUM_0
      #define WAKEUP_LEVEL 0
      #define INTERFACE_COUNT 1
      #define OCP_TUNED 0x38

      const int pin_btn_usr1 = 0;

      #if defined(EXTERNAL_LEDS)
        const int pin_led_rx = 13;
        const int pin_led_tx = 14;
      #else
        const int pin_led_rx = 35;
        const int pin_led_tx = 35;
      #endif

      const uint8_t interfaces[INTERFACE_COUNT] = {SX1262};
      const bool interface_cfg[INTERFACE_COUNT][3] = { 
                    // SX1262
          {
              true, // DEFAULT_SPI
              true, // HAS_TCXO
              true  // DIO2_AS_RF_SWITCH
          }, 
      };
      const int8_t interface_pins[INTERFACE_COUNT][10] = { 
                  // SX1262
          {
              8, // pin_ss
              9, // pin_sclk
              10, // pin_mosi
              11, // pin_miso
              13, // pin_busy
              14, // pin_dio
              12, // pin_reset
              -1, // pin_txen
              -1, // pin_rxen
              -1  // pin_tcxo_enable
          }
      };

If we want to add support for a new board we have to add a new section for this board. Here we define all the features it has or not has.

## Define the ESP32-S3-N16R8 with the E22-400M33S board for real

First we have to make define both BOARD_MODEL and BOARD_VARIANT.
Let's define BOARD_MODEL as 0x62 and BOARD_VARIANT as 0x32 as these numbers are not used yet. 
Here the 0x32 indicates the use of a SX1268 Semtech E22 LoRa transceiver.

$ sudo nano Makefile

    # Added-board from PA2EON
    firmware-espressif-esp32-s3-n16r8: check_bt_buffers
       arduino-cli compile --fqbn "esp32:esp32:esp32s3:CDCOnBoot=cdc" $(COMMON_BUILD_FLAGS) --build-property          "compiler.cpp.extra_flags=\"-DBOARD_MODEL=0x62\""  

    # Added board from PA2EON
    upload-espressif-esp32-s3-n16r8:
	    arduino-cli upload -p $(or $(port), /dev/ttyACM0) --fqbn esp32:esp32:esp32s3
	    @sleep 1
	    rnodeconf $(or $(port), /dev/ttyACM0) --firmware-hash $$(./partition_hashes ./build/esp32.esp32.esp32s3/RNode_Firmware_CE.ino.bin)
	    @sleep 3
	    python3 ./Release/esptool/esptool.py --port $(or $(port), /dev/ttyACM0) --chip esp32-s3 --baud 921600 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size 4MB 0x210000 ./Release/console_image.bin

    # Added board from PA2EON
    release-espressif-esp32-s3-n16r8: check_bt_buffers
        arduino-cli compile --fqbn "esp32:esp32:esp32s3:CDCOnBoot=cdc" $(COMMON_BUILD_FLAGS) --build-property "compiler.cpp.extra_flags=\"-DBOARD_MODEL=0x62\" \"-DBOARD_VARIANT=0x32\""
        cp ~/.arduino15/packages/esp32/hardware/esp32/$(ARDUINO_ESP_CORE_VER)/tools/partitions/boot_app0.bin build/rnode_firmware_espressif-esp32-s3-n16r8.boot_app0
        cp build/esp32.esp32.ttgo-lora32/RNode_Firmware_CE.ino.bin build/rnode_firmware_espressif-esp32-s3-n16r8.bin
        cp build/esp32.esp32.ttgo-lora32/RNode_Firmware_CE.ino.bootloader.bin build/rnode_firmware_espressif-esp32-s3-n16r8.bootloader
        cp build/esp32.esp32.ttgo-lora32/RNode_Firmware_CE.ino.partitions.bin build/rnode_firmware_espressif-esp32-s3-n16r8.partitions
        zip --junk-paths ./Release/rnode_firmware_espressif-esp32-s3-n16r8.zip ./Release/esptool/esptool.py ./Release/console_image.bin build/rnode_firmware_t3s3_sx126xrnode_firmware_espressif-esp32-s3-n16r8.boot_app0 build/rnode_firmware_espressif-esp32-s3-n16r8.bin build/rnode_firmware_espressif-esp32-s3-n16r8.bootloader build/rnode_firmware_espressif-esp32-s3-n16r8.partitions
        rm -r build
	
 In the file <b>Boards.h</b> make the next an entries for the new board within the '#if MCU_VARIANT == MCU_ESP32':

     // Board added by PA2EON
     #define BOARD_ESPRESSIF_ESP32_S3_n16r8 0x62 // Espressif ESP32-S3 N16R8
     #define MODEL_32            0x32 // SX1268
     // Place above lines abov the 'Displays' item
      
      #elif BOARD_MODEL == BOARD_ESPRESSIF_ESP32_S3_n16r8
      #define IS_ESP32S3 true
      #define HAS_DISPLAY false
      #define HAS_BLUETOOTH false
      #define HAS_BLE true
      #define HAS_PMU true
      #define HAS_CONSOLE true
      #define HAS_EEPROM true
      //#define HAS_INPUT true
      //#define HAS_SLEEP true
      //#define PIN_WAKEUP GPIO_NUM_0
      //#define WAKEUP_LEVEL 0
      #define INTERFACE_COUNT 1
      #define OCP_TUNED 0x38

      const int pin_btn_usr1 = 0;

      #define HAS_NP true
      const int pin_np = 21;
      #if HAS_NP == false
        #if defined(EXTERNAL_LEDS)
          const int pin_led_rx = 16;
          const int pin_led_tx = 17;
        #else
          const int pin_led_rx = 21;
          const int pin_led_tx = 21;
        #endif
      #endif

      const uint8_t interfaces[INTERFACE_COUNT] = {SX1268};
      const bool interface_cfg[INTERFACE_COUNT][3] = {
                    // SX1268
          {
              false, // DEFAULT_SPI
              false, // HAS_TCXO
              false  // DIO2_AS_RF_SWITCH
          },
      };
      const uint8_t interface_pins[INTERFACE_COUNT][10] = {
                  // SX1268
          {
               10, // pin_ss
               12, // pin_sclk
               11, // pin_mosi
               13, // pin_miso
               -1, // pin_busy
               15, // pin_dio
               14, // pin_reset
               -1, // pin_txen
               -1, // pin_rxen
               -1  // pin_tcxo_enable
          }
      };

      // Place above text before the lines ' unsupported ESP32 boards ... ' 

Also, in the file Utilities.h add entry:

	#elif BOARD_MODEL == BOARD_ESPRESSIF_ESP32_S3_n16r8
		void led_rx_on()  { digitalWrite(pin_led_rx, HIGH); }
		void led_rx_off() { digitalWrite(pin_led_rx, LOW); }
		void led_tx_on()  { digitalWrite(pin_led_tx, HIGH); }
		void led_tx_off() { digitalWrite(pin_led_tx, LOW); }
		void led_id_on()  { }
		void led_id_off() { }

When al above is added to the files we can flash the firmware;

### Flashing the board

Make shure you are in the dialout group. If not:

    $ sudo usermod -aG dialout <username>

If you are flashing a custom board, you will need to generate a signing key in rnodeconf prior to flashing if you do not already have one by running:

    $ rnodeconf -k

Then flash the firmware;

    $ make firmware-espressif-esp32-s3-n16r8
  


