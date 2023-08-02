# Installing Digilent Adept

OpenOCD does not use Digilent drivers to communicate with Digilent
debug cables. It uses it's own implementation of FTDI MPSSE protocol,
which is compatible with FTDI x232 cable. However, you might need to install
Digilent Adept drivers and utilities to use some features of Digilent cables.

Download an appropriate version of runtime and utilities from
[Digilent Adept download page](https://digilent.com/shop/software/digilent-adept/download).
There several options:

* **Adept for Windows System** — utilities and runtime in a single installer for Windows.
* **Adept for Linux Runtime** — runtime for Linux. `rpm` and `dep` packages
  are available, as well as a generic `.tar.gz` archive.
* **Adept Utilities** — utilities for Linux. `rpm` and `dep` packages
  are available, as well as a generic `.tar.gz` archive.

You can check, that utilities and runtime are installed correctly
using `djtgcfg` utility:

```text
$ djtgcfg enum
    Found 1 device(s)

    Device: JtagHs2
        Product Name:   Digilent JTAG-HS2
        User Name:      JtagHs2
        Serial Number:  210249810909
```
