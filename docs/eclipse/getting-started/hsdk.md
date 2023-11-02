# Getting Started with HS Development Kit

!!! info

    Consider reading these guides firstly:

    * [Configuring HS Development Kit](../../platforms/board-hsdk.md)
    * [Configuring HS Development Kit 4xD](../../platforms/board-hsdk-4xd.md)
    * [Installing WinUSB driver on Windows](../../platforms/winusb.md)

## Creating the Project

Select **File** → **New** → **Project..** and choose **C Project**.
A list of ARC projects will appear. Choose any **ARC HS Development Kit Hello World Project**
from the **ARC HS Development Kit Projects** group:

![HS Development Kit Projects](./images/hsdk-projects.png)

An empty project will be created. Add a new C source file with name `main.c`
in `src` directory in **Project Explorer**:

```c
/* Print a greeting on UART output and exit. */

#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("Hello, World!\n\r");
    return 0;
}
```

## Building the Project

Do right click on a project in **Project Explorer** ans choose **Build Project**.
The project will be built with this output:

```text
make all 
Building file: ../src/main.c
Invoking: ARC GNU C Compiler
arc-elf32-gcc -mcpu=hs38_linux -matomic -mll64 -mdiv-rem -mswap -mnorm -mmpy-option=9 -mbarrel-shifter -mfpu=fpud_all --param l1-cache-size=65536 --param l1-cache-line-size=64 -include  /home/ykolerov/runtime-EclipseApplication/hsdk_hello/Debug/core_config.h -O0 -g3 -Wall -c -fmessage-length=0 -gdwarf-2 -Wa,-adhlns="src/main.o.lst" -MMD -MP -MF"src/main.d" -MT"src/main.o" -o "src/main.o" "../src/main.c"
Finished building: ../src/main.c
 
Building target: hsdk_hello.elf
Invoking: ARC GNU C Linker
arc-elf32-gcc -mcpu=hs38_linux -matomic -mll64 -mdiv-rem -mswap -mnorm -mmpy-option=9 -mbarrel-shifter -mfpu=fpud_all --param l1-cache-size=65536 --param l1-cache-line-size=64 -specs=hsdk.specs -Wl,-Map,hsdk_hello.map -o "hsdk_hello.elf" ./src/main.o 
/home/ykolerov/workspace/newlib-memory-map/arc-gnu-toolchain/arc-elf32/lib/gcc/arc-elf32/13.1.1/../../../../arc-elf32/bin/ld: warning: hsdk_hello.elf has a LOAD segment with RWX permissions
Finished building target: hsdk_hello.elf
```

## Creating a Debug Configuration

Do right click on projects's name in **Project Explorer** and choose
**Debug As** → **Debug Configurations...**. Then do right click on
**ARC C/C++ application** and choose **New Configuration**. Here is a main window of
the debug configuration:

![Debug Configuration - Main](./images/hsdk-debug-conf-main.png)

Ensure that a correct project and binary are selected. Navigate to **Main** tab
and **Gdbserver Settings** inner tab:

![Debug Configuration - GDB](./images/hsdk-debug-conf-gdb.png)

Choose **JTAG via OpenOCD** as ARC GDB Server and **HS Development Kit** as
a development system (use a corresponding one for your case). Also,
choose **HS38 #1** target core. Then click on **Apply**.

## Configuring a Serial Terminal

Navigate to **Terminal** inner tab of **Main** tab and select a COM port for
the board. Eclipse automatically detects all available COM ports. In my case
it's `COM7`.

![Debug Configuration - Terminal](./images/hsdk-debug-conf-terminal.png)

On Windows you can find the exact number in **Device Manager** (it corresponds
to **USB Serial Port** device):

![Debug Configuration - Port](./images/emsk-debug-conf-port.png)

On Linux a serial device for HS Development Kit is usually `/dev/ttyUSB0`
if there are no other serial devices connected to the host.

## Debugging the Project

Open the debug configuration in **Debug Configurations** windows and click
on **Debug** button. The **Debug** perspective will be opened:

![Debug - Perspective](./images/hsdk-debug-perspective.png)

Use **Step Over** button to step over `printf` function and select **Terminal**
in the bottom of the window. "Hello world!" string will be printed:

![Debug - Output](./images/hsdk-debug-output.png)
