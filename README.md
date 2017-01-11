Solidrun Braswell SOM Coreboot
==============================

How to build:

Prepare the environment:

    git clone --recursive https://github.com/coreboot/coreboot.git
    cd coreboot
    patch -p1 < ../solidrun.patch
    mkdir -p 3rdparty/blobs/mainboard/solidrun/braswell_som
    cp -rf ../solidrun_braswell_blobs/* 3rdparty/blobs/mainboard/solidrun/braswell_som
    cp config .config

Build the cross toolchain:

    make crosstools-x64

Configure payload and build:

    make menuconfig
    <choose payload>
    make -j8

Flash the resulting `build/coreboot.rom`.

CAREFUL: Do not exceed a voltage of 1.8V while flashing the SPI Flash. This will damage your board!

Note: You can use the supplied UEFIPAYLOAD.fd as a TianoCore payload. This was built from the [EDK-II repository](https://github.com/tianocore/edk2) with the `CorebootPayloadPkg`.

The code was developed against git revision `8e641741377e2d11b23ced0a91d1e5cd66ed7434`. The code still compiles against the current master but is currently untested. It should still work. If in doubt, the to check out the specific git revision mentioned above.

All the important coreboot related code is in `src/mainboard/solidrun`

Details about the supplied blobs
--------------------------------

* BSWFSP.fd
    * Intel Braswell FSP relocated to baseaddress 0xfff6e000 using the Intel Binary Configuration Tool supplied with the Braswell FSP
    * http://www.intel.de/content/www/de/de/intelligent-systems/intel-firmware-support-package/intel-fsp-overview.html

* SEC_Region.bin
    * Intel TXT region carved out of the supplied UEFI image
    * There are also certain russian websites where whole archives with different versions of the TXT firmwares are offered
    * Can be configured with "Intel TXE System Tools" which are under NDA. Google may help.

* VbtBswRvp.bin
    * Display driver configuration binary supplied by board manufacturer
    * Also carved out of the supplied UEFI binary when debugging non-working graphics output - they are identical

* descriptor.bin
    * Intel flash descriptor describing the flash contents and soft-straps
    * Taken from the original flash image
    * Can be modified with an Intel internal tool called "Flash Image Tool". Contained in the "Intel TXE System Tools". Google.

* microcode.bin
    * Intel Microcode carved out of the supplied UEFI image. Might not be the newest version and might not contain all available steppings for all supported CPUs. Intel doesn't seem to offer microcode for Braswell CPUs publicly.

Known issues
------------

* GPIO config currently consists only of register dumps instead of macros like for other Braswell based boards. The whole file needs cleanup
* Windows doesn't boot. The kernel BUGCHECKs with ACPI_OS_ERROR
* This build of coreboot was only tested against the smaller Atom E8000 CPU
* If no displays are attached on bootup, the Intel FSP does not initialize graphics output
* The Intel FSP only supports the UEFI Graphics Output Protocol for graphics related things, SeaBIOS would need a VGA-BIOS, so SeaBIOS doesn't have any graphics output at all
* The only working memory configuration currently is 4GiB, because I could't figure out the resistor strapping. All necessary SPDs for memory config are present
* Currently all devices are enabled. All UARTs, all I2C interfaces etc. This might need some cleanup
* I have no expericence with ACPI. There might be some errors in the tables (and the fact that Windows isn't booting)
* There is some general cleanup needed in a few files to be able to upstream the board support
