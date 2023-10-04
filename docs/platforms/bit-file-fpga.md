# Writing Bit-file into FPGA with Digilent HS Cable

Below is an example given of how to program a Xilinx ML509 FPGA developers
board. NOTE: make sure the Digilent HS cable is connected to the right
JTAG connector on the board (programming the FPGA and not the memory).
So, it should be connected to PC4 JTAG and not to J51 BDM.
Further more, Device 4: XC5VLX110T is the FPGA to program, device 4 in the
JTAG scan chain.

```text
> djtgcfg enum
Found 1 device(s)

Device: JtagHs2
    Product Name:   Digilent JTAG-HS2
    User Name:      JtagHs2
    Serial Number:  210249810909

> djtgcfg -d JtagHs2 init
Initializing scan chain...
Found Device ID: a2ad6093
Found Device ID: 0a001093
Found Device ID: 59608093
Found Device ID: f5059093
Found Device ID: f5059093

Found 5 device(s):
    Device 0: XCF32P
    Device 1: XCF32P
    Device 2: XC95144XL
    Device 3: XCCACE
    Device 4: XC5VLX110T

> djtgcfg -d JtagHs2 prog -i 4 -f <fpga bit file to progam>.bit
Programming device. Do not touch your board. This may take a few minutes...
Programming succeeded.
```
