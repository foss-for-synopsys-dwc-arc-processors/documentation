# General Topics

## Preface

* Refer to [Development Platforms Table](#development-platforms-table) section
  for options for all available configurations of the board.
* Follow [Memory Maps and Linker Scripts](./memory.md) guide for details
  about `memory.x` files and where they may be obtained.
* Follow a [corresponding manual](../../platforms/get-openocd.md) to obtain
  and configure OpenOCD.

## Connecting Using UART

Connecting a board to the host computer allows you using a serial port terminal for interacting with the board.
You can use any software to connect to the serial port terminal:

* On Windows you can use HyperTerminal or Putty.
* On Linux you can use `minicom`, `gtkterm`, `cutecom`, etc.

You can find detailed instructions for particular boards in corresponding sections.

## Development Platforms Table

| Board       | Core       | `-specs=`     | Memory map        | OpenOCD config        | GCC options                                                                  |
|-------------|------------|---------------|-------------------|-----------------------|------------------------------------------------------------------------------|
| EMSK 1      | EM4        | `emsk.specs`  | `emsk1_em4.x`     | `snps_em_sk_v1.cfg`   | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |
| EMSK 1      | EM6        | `emsk.specs`  | `emsk1_em6.x`     | `snps_em_sk_v1.cfg`   | `-mcpu=em4_dmips -mmpy-option=wlh5`                                          |
| EMSK 2.0    | EM5D       | `emsk.specs`  | `emsk2.1_em5d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| EMSK 2.0    | EM7D       | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter`                 |
| EMSK 2.0    | EM7DFPU    | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4 -mswap -mnorm -mmpy-option=wlh3 -mbarrel-shifter -mfpu=fpuda_all` |
| EMSK 2.1    | EM5D       | `emsk.specs`  | `emsk2.1_em5d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| EMSK 2.1    | EM7D       | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_dmips -mmpy-option=wlh3`                                          |
| EMSK 2.1    | EM7DFPU    | `emsk.specs`  | `emsk2.1_em7d.x`  | `snps_em_sk_v2.1.cfg` | `-mcpu=em4_fpuda -mmpy-option=wlh3`                                          |
| EMSK 2.2    | EM7D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_dmips`                                                            |
| EMSK 2.2    | EM9D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_fpus -mfpu=fpus_all`                                              |
| EMSK 2.2    | EM11D      | `emsk.specs`  | `emsk2.2_em11d.x` | `snps_em_sk_v2.2.cfg` | `-mcpu=em4_fpuda -mfpu=fpuda_all`                                            |
| EMSK 2.3    | EM7D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_dmips`                                                            |
| EMSK 2.3    | EM9D       | `emsk.specs`  | `emsk2.2_em9d.x`  | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_fpus -mfpu=fpus_all`                                              |
| EMSK 2.3    | EM11D      | `emsk.specs`  | `emsk2.2_em11d.x` | `snps_em_sk_v2.3.cfg` | `-mcpu=em4_fpuda -mfpu=fpuda_all`                                            |
| HSDK        | HS3x       | `hsdk.specs`  | `hsdk.x`          | `snps_hsdk.cfg`       | `-mcpu=hs38_linux`                                                           |
| HSDK 4x/4xD | HS4x/HS4xD | `hsdk.specs`  | `hsdk.x`          | `snps_hsdk_4xd.cfg`   | `-mcpu=hs38_linux`                                                           |
| IoTDK       | ???        | `iotdk.specs` | `iotdk.x`         | `snps_iotdk.cfg`      | `-mcpu=em4_dmips`                                                            |
