# Resetting Boards

## Using `rff-reset`

You can use the `rff-reset` utility to reset ARC development boards.
Follow instructions in the [`rff-rtdi-reset` repository](https://github.com/foss-for-synopsys-dwc-arc-processors/rff-ftdi-reset)
to install and use the utility.

Here is an example for a case when only one board is connected to the host:

```text
# rff-reset --only-one
```

Here is an example for a case when it's necessary to reset a particular
board ([Digilent Adept utility](./digilent.md) is used to obtain a serial number):

```text
# djtgcfg enum
Found 1 device(s)

Device: HSDK-4xD
    Device Transport Type: 00020001 (USB)
    Product Name:          DesignWare ARC SDP
    User Name:             HSDK-4xD
    Serial Number:         251642000213

# rff-reset --serial 251642000213
```

## Using OpenOCD

It is possible to reset ARC SDP board without touching the physical button
on the board. This can be done using an OpenOCD script:

```shell
$ openocd -f test/arc/reset_sdp.tcl
```

Note that OpenOCD will crash with a segmentation fault after executing this
script - this is expected and happens only after board has been reset, but
that means that other OpenOCD scripts cannot be used in chain with
`reset_sdp.tcl`, first OpenOCD should be invoked to reset the board,
second it should be invoked to run as an actual debugger.
