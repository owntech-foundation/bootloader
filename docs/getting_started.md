
# Bootloader

OwnTech use its own bootloader in order to be able to flash code using USB without the use of a dedicated debug tool. 

It is based on [MCUBoot](https://docs.mcuboot.com/) an open source bootloader supported by the zephyr ecosystem. 

# Features

- USB ready
- Supports Over The Air (OTA) updates
- Image validation 
- Encryption ready

# Boot sequence

<div class="grid cards" markdown>

- ![Memory_map](bootloader_boot.drawio.svg){ align=left }{ .card }


When pressing reset button the following happens : 
- Program initiates at 0x80000000, and bootloader is launched.
- Bootloader jumps at Image-0 address at 0x8010000. { .card }

</div>


# Nomal Upload sequence

=== "Upload"
    ![Memory_map](bootloader.drawio.svg){ align=left }

    When pressing upload button the following happens : 

    - Image-1 is the memory bank where the program is uploaded using USB. 
    - When the USB transfer is complete, a reboot order is sent to the board. 
    - Image is marked for testing, so a memory swap is performed. Image-1 is sent to memory bank 0 at the address 0x8010000 and Image-0 is sent to memory bank 1 at the address 0x8047800. 
    - A boot test is performed. If the initialization is successful, the new program is marked as good and stays in image-0. Otherwise, the image is rejected and the swap action is reverted. 
    - Application code is executed from address 0x8010000


!!! warning

    Image swapping requires having a valid image at address 0x8010000. 
    If no image-0 is present normal upload sequence will fail. Use Recovery Mode to upload a valid Image-0 and proceed again.

# Recovery Mode 

=== "Recovery"
    ![Memory_map](bootloader_recovery.drawio.svg){ align=left }

    The OwnTech bootloader has a recovery mode in order to flash directly the Image-0 without performing a swap action. 

    To enter recovery mode, press BOOT button and RESET button simultaneously. 

    When pressing the upload button in recovery mode the following happens : 

    - User program is written at the address 0x8010000 directly. 
    - A reboot is performed and the bootloader jumps to user code at address 0x8010000.

!!! note
    
    When entering recovery mode, the user LED should light up

!!! note
    
    Recovery mode is significantly slower than Normal Upload Sequence.


# How it works

USB upload uses what is called a magic baudrate callback. 

When pressing the upload button:

- The user code is compiled, creating a bin executable.
- A trailer containing meta-data is added to the bin file, and the executable is marked for testing.  
- The USB serial disconnects and reconnects using the magic 1200Baud baudrate. 
- That baudrate is detected and it triggers a reboot order to switch in bootloader mode.
- On the computer side, a small program called MCUMgr is called to upload the user code image to the microcontroller (it is located in owntech/third_party/mcumgr - if it is not present, it will be downloaded automatically.)
- When upload is finished, a reboot order is sent
- Bootloader starts and detects the new image trailer marked for testing.
- Bootloader performs the swap action
- Bootloader jumps to user application at address 0x8010000 and USB serial is available again.