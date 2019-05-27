= ASUS H110M mainboard

This commodity board mostly works with LinuxBoot, although it has some
quirks.  USB works.  Ethernet shows up in `lspci`.

== To-Do

Things to do:

* Fix `DxeCore` per-file LZMA issue
* Remove microcode from FIT
* Leave SMM unlocked
* Leave memory controller unlocked
* Reclaim extra space from copy of PEI

== Quirks
There are some issues with the AMI APTIO ROM image:

* The SuperIO chip redirects port 0x80 to the serial port
* Per-file LZMA compression doesn't work with stock edk2
* Firmware Volume LZMA compression doesn't work with their `PeiCore`
* The `DxeCore` can't be replaced due to compression issues
* PEI firmware hasn't been minimized
* BootGuard is not enabled

The weirdest one is that the SuperIO chips routes the bytes
written to the POST io port 0x80 as bytes to the serial port.
This means that there is always garbage on the port during the
boot and that the `PeiCore` or `DxeCore` can't output any messages.
Something in the DXE volume converts it back to normal serial I/O 
before the hand off to the BDS, so we see the messages at the
LinuxBoot handoff.

We want to replace the DxeCore and SmmCore with our own
but there are issues with the supported compression formats.
AMI has per-file LZMA compression enabled, which doesn't work
with stock-edk2 `DxeCore`.  We can work around the serial port
weirdness, but this compression prevents us from replacing
the `DxeCore` and `SmmCore`.

There are two copies of PEI; we might be able to reclaim 2 MB
of flash space by removing the extra firmware volumes.  There is
also a few hundred KB that could be reclaimed from the VGA BIOS,
Logo and Font volumes.  Both of these changes would need to
adjust layout in PEI.

