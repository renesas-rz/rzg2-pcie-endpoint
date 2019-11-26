# RZ/G2 PCIe Endpoint Controller Driver

The PCIe controller IP in RZ/G2 are capable of operating either in Root Complex
mode (host) or Endpoint mode (device).

When operating in endpoint mode, the controller can be configured to be used as
any function depending on the use case. ("Test endpoint" is the only PCIe EP
function supported in Linux kernel right now).

The endpoint driver supports legacy interrupt mode.

The repository provides patches that apply on top of Linux v5.4.

## Source Prerequisites
1. Linux version [v5.4](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/log/?h=v5.4)
2. PCIe-ep support patches for G2E

## Hardware Modifications
The HiHope RZ/G2[MN] and Si-Linux RZ/G2E platforms will not work in endpoint
mode without some modifications.

The image below shows the modifications that need to be done on the Si-Linux
RZ/G2E board being used as the endpoint:
* Remove U11 (9fgv0441) from the board
* Use a wire to connect the pads from pin 13 (DIF0) to pin 18 (DIF1).
* Use a wire to connect the pads from pin 14 (DIF0#) to pin 19 (DIF1#).

![Hardware_mod-g2e.png](https://github.com/renesas-rz/rzg2-pcie-endpoint/raw/linux-5.4/Hardware_mod-g2e.png "Hardware_mod-g2e.png")


To connect the endpoint board to the RC board, the pins shown in the below table
need to be connected. Note that not all pins need to be connected.

| Side  | # | Name | Description | RZ/G2 SubBoard (Root) | | Side | # |  Name | RZ/G2 SubBoard (Endpoint) |
|--|--|--|--|--|--|--|--|--|--|
| A | 1 | PRSNT#1 | Hot plug presence detect | Ground | wire to | B | 17 | PRSNT#2 | PU to VDD3.3V |
| A | 2 | +12v | +12 volt power | D12.0V |
| A | 3 | +12v | +12 volt power | D12.0V |
| A | 4 | GND | Ground | Ground |
| A | 5 | JTAG2 | TCK| PD |
| A | 6 | JTAG3 | TDI| PU to VDD3.3V |
| A | 7 | JTAG4 | TDO| NC |
| A | 8 | JTAG5 | TMS| PU to VDD3.3V |
| A | 9 | +3.3v | +3.3 volt power | VDD3.3V |
| A | 10 | +3.3v | +3.3 volt power | VDD3.3V |
| A | 11 | PERST# | Reset/Power Good | PRESETOUTn (3.3V, from RZ/G2M) |
| A | | | Mechanical Key | |
| A | 12 | GND | Ground | Ground | wire to | A | 12 | GND | Ground |
| A | 13 | REFCLK+ | Reference Clock | PCIE0_CN_CLK_P (output from U1301) | wire to | A | 13 | REFCLK+ | PCIE0_CN_CLK_P (MODIFIED: to RZ/G2M) |
| A | 14 | REFCLK- | Differential pair | PCIE0_CN_CLK_M (output from U1301) | wire to | A | 14 | REFCLK- | PCIE0_CN_CLK_M (MODIFIED: to RZ/G2M) |
| A | 15 | GND | Ground | Ground | wire to | B | 13 | GND | Ground |
| A | 16 | HSIp(0) | Receiver Lane 0, | PCIE0_RX_P (to RZ/G2M) | wire to | B | 14 | HSOp(0) | PCIE0_TX_P (from RZ/G2M via cap) |
| A | 17 | HSIn(0) | Differential pair | PCIE0_RX_M (to RZ/G2M) | wire to | B | 15 | HSOn(0) | PCIE0_TX_M (from RZ/G2M via cap) |
| A | 18 | GND | Ground | Ground | wire to | B | 16 | gnd | Ground |
| B | 1 | +12v | +12 volt power | D12.0V |
| B | 2 | +12v | +12 volt power | D12.0V |
| B | 3 | +12v | +12 volt power | NC (R2488 to D12.0V) |
| B | 4 | GND | Ground | Ground |
| B | 5 | SMCLK | SMBus clock | PU to VDD3.3V |
| B | 6 | SMDAT | SMBus data | PU to VDD3.3V |
| B | 7 | GND | Ground | Ground |
| B | 8 | +3.3v | +3.3 volt power | VDD3.3V |
| B | 9 | JTAG1 | +TRST# | PD |
| B | 10 | 3.3Vaux | 3.3v volt power | NC (R1205 to VDD3.3V) |
| B | 11 | WAKE# | Link Reactivation | PU to VDD3.3V |
| B | | | Mechanical Key | |
| B | 12 | RSVD| Reserved | NC |
| B | 13 | GND | Ground | Ground | wire to | A | 15 | GND | Ground |
| B | 14 | HSOp(0) | Transmitter Lane 0, | PCIE0_TX_P (from RZ/G2M via cap) | wire to | A | 16 | HSIp(0) | PCIE0_RX_P (to RZ/G2M) |
| B | 15 | HSOn(0) | Differential pair | PCIE0_TX_M (from RZ/G2M via cap) | wire to | A | 17 | HSIn(0) | PCIE0_RX_M (to RZ/G2M) |
| B | 16 | GND | Ground | Ground | wire to | A | 18 | GND | Ground |
| B | 17 | PRSNT#2 | Hotplug detect | PU to VDD3.3V | wire to | A | 1 | PRSNT#1 | Ground |
| B | 18 | GND | Ground | Ground |

The image below shows an example of how this can be done.

![pcie-ep-cable.png](https://github.com/renesas-rz/rzg2-pcie-endpoint/raw/linux-5.4/pcie-ep-cable.png "pcie-ep-cable.png")

## Host PC Tools
1. Terminal software, such as TeraTerm or HyperTerminal on Windows, to see the
output from and interact with the U-Boot boot loader and the Linux Kernel.

## Preconditions
1. Serial terminal configured for 115200 baud, 8-bit data, no parity, 1-bit
   stop, no flow control.
2. Hardware modifications required for board to work as endpoint as per the table above.
3. pcitest utility should be part Rootfs/NFS for board working as RC.

## Driver Configuration for EP
The following config options have to be enabled in the Kernel in order to
configure the PCI controller to be used as an "Endpoint Test" function driver.

1. `CONFIG_PCI_ENDPOINT=y`
2. `CONFIG_PCI_EPF_TEST=y`
3. `CONFIG_PCIE_RCAR_EP=y`

## Driver Configuration for RC
The following config options have to be enabled in the Kernel in order to
configure the PCI controller to be used as RC.

1. `CONFIG_PCI_ENDPOINT_TEST=y`
2. `CONFIG_PCIE_RCAR_HOST=y`

Note: `CONFIG_PCIE_RCAR_HOST` is a config option for the RCAR PCIe controller
driver, if using a different controller enable the specific config option.

## Endpoint setup
To find the list of endpoint controller devices in the system:
```
$ ls /sys/class/pci_epc/
fe000000.pcie_ep
```

To find the list of endpoint function drivers in the system:
```
$ ls /sys/bus/pci-epf/drivers
pci_epf_test
```

### Creating pci-epf-test device
The PCIe endpoint function device can be created using configfs. To create a
pci-epf-test device, the following commands can be used:
```
$ mount -t configfs none /sys/kernel/config
$ cd /sys/kernel/config/pci_ep
$ mkdir functions/pci_epf_test/func1
```

The `mkdir functions/pci_epf_test/func1` above creates the pci-epf-test function
device.

The PCIe endpoint framework populates the directory with configurable fields:
```
$ cd /sys/kernel/config/pci_ep
$ ls functions/pci_epf_test/func1/
baseclass_code  cache_line_size  deviceid interrupt_pin  msi_interrupts
msix_interrupts  progif_code  revid  subclass_code  subsys_id  subsys_vendor_id
vendorid
```

The driver populates these entries with default values when the device is bound
to the driver. The pci-epf-test driver populates vendorid with 0xffff and
interrupt_pin with 0x0001.
```
$ cd /sys/kernel/config/pci_ep/functions/pci_epf_test/func1/
$ cat vendorid
0xffff
$ cat interrupt_pin
0x0001
```

### Creating pci-epf-test device
The user can configure the pci-epf-test device using the configfs. In order to
change the vendorid and device-id, the following commands can be used:
```
$ cd /sys/kernel/config/pci_ep
# Configure Renesas as the vendor for G2E
$ echo 0x1912 > functions/pci_epf_test/func1/vendorid
# Configure the device ID
$ echo 0x002d > functions/pci_epf_test/func1/deviceid
# Set number of MSI interrupts
$ echo 32 > functions/pci_epf_test/func1/msi_interrupts
```

### Binding pci-epf-test device to a EP controller
```
$ cd /sys/kernel/config/pci_ep
$ ln -s functions/pci_epf_test/func1/ controllers/fe000000.pcie_ep/
```

### Starting the EP device
```
$ cd /sys/kernel/config/pci_ep
$ echo 1 > controllers/fe000000.pcie_ep/start
```

## RC Setup
The PCIe EP device must be powered-on and configured before the PCIe HOST
device. This restriction is because the PCIe HOST driver doesn’t have hot plug
support.

```
$ lspci -vvxx
01:00.0 Unassigned class [ff00]: Renesas Technology Corp. Device 002d
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0
        Interrupt: pin A routed to IRQ 114
        Region 0: Memory at fe200200 (64-bit, non-prefetchable) [size=128]
        Region 2: Memory at fe200000 (64-bit, non-prefetchable) [size=256]
        Region 4: Memory at fe200100 (64-bit, non-prefetchable) [size=256]
        Capabilities: [40] Power Management version 3
                Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0+,D1-,D2-,D3hot+,D3cold+)
                Status: D0 NoSoftRst- PME-Enable- DSel=0 DScale=0 PME-
        Capabilities: [50] MSI: Enable+ Count=1/1 Maskable+ 64bit+
                Address: 00000004f9c62000  Data: 0001
                Masking: 00000000  Pending: 00000000
        Capabilities: [70] Express (v2) Endpoint, MSI 00
                DevCap: MaxPayload 128 bytes, PhantFunc 0, Latency L0s unlimited, L1 unlimited
                        ExtTag+ AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 0.000W
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag+ PhantFunc- AuxPwr- NoSnoop+
                        MaxPayload 128 bytes, MaxReadReq 128 bytes
                DevSta: CorrErr+ UncorrErr+ FatalErr- UnsuppReq+ AuxPwr- TransPend-
                LnkCap: Port #0, Speed 5GT/s, Width x1, ASPM L0s, Exit Latency L0s unlimited
                        ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp-
                LnkCtl: ASPM Disabled; RCB 64 bytes Disabled- CommClk-
                        ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
                LnkSta: Speed 5GT/s, Width x1, TrErr- Train- SlotClk- DLActive- BWMgmt- ABWMgmt-
                DevCap2: Completion Timeout: Not Supported, TimeoutDis+, LTR-, OBFF Not Supported
                         AtomicOpsCap: 32bit- 64bit- 128bitCAS-
                DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
                         AtomicOpsCtl: ReqEn-
                LnkCtl2: Target Link Speed: 5GT/s, EnterCompliance- SpeedDis-
                         Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
                         Compliance De-emphasis: -6dB
                LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete-, EqualizationPhase1-
                         EqualizationPhase2-, EqualizationPhase3-, LinkEqualizationRequest-
        Capabilities: [100 v1] Virtual Channel
                Caps:   LPEVC=0 RefClk=100ns PATEntryBits=1
                Arb:    Fixed- WRR32- WRR64- WRR128-
                Ctrl:   ArbSelect=Fixed
                Status: InProgress-
                VC0:    Caps:   PATOffset=00 MaxTimeSlots=1 RejSnoopTrans-
                        Arb:    Fixed- WRR32- WRR64- WRR128- TWRR128- WRR256-
                        Ctrl:   Enable+ ID=0 ArbSelect=Fixed TC/VC=ff
                        Status: NegoPending- InProgress-
        Kernel driver in use: pci-endpoint-test
00: 12 19 2d 00 06 04 10 00 00 00 00 ff 00 00 00 00
10: 04 02 20 fe 00 00 00 00 04 00 20 fe 00 00 00 00
20: 04 01 20 fe 00 00 00 00 00 00 00 00 00 00 00 00
30: 00 00 00 00 40 00 00 00 00 00 00 00 67 01 00 00
```

## PCITEST utility
```
$ pcitest  -h
usage: pcitest [options]
Options:
        -D <dev>                PCI endpoint test device {default: /dev/pci-endpoint-test.0}
        -b <bar num>            BAR test (bar number between 0..5)
        -m <msi num>            MSI test (msi number between 1..32)
        -x <msix num>           MSI-X test (msix number between 1..2048)
        -i <irq type>           Set IRQ type (0 - Legacy, 1 - MSI, 2 - MSI-X)
        -I                      Get current IRQ type configured
        -l                      Legacy IRQ test
        -r                      Read buffer test
        -w                      Write buffer test
        -c                      Copy buffer test
        -s <size>               Size of buffer {default: 100KB}
        -h                      Print this help message
```

### BAR test
```
$ pcitest -b 0
BAR0:           OKAY
$ pcitest -b 2
BAR2:           OKAY
$ pcitest -b 4
BAR4:           OKAY
```

### Set Legacy IRQ
```
$ pcitest -i 0
SET IRQ TYPE TO LEGACY:         OKAY
```

### Test Legacy IRQ
```
$ pcitest -i 0
LEGACY IRQ:     OKAY
```

### Read Test
```
$ pcitest -r 1024
READ ( 102400 bytes):           OKAY
```

### Write Test
```
$ pcitest -w 1024
WRITE ( 102400 bytes):          OKAY
```

### Copy Test
```
$ pcitest -c 1024
COPY ( 102400 bytes):           OKAY
```

---

This project is licensed under the terms of the GPLv2 license.

Send pull requests, patches, comments or questions to:  
Chris Paterson <chris.paterson2@renesas.com>.

Maintainer:  
Chris Paterson <chris.paterson2@renesas.com>.

Note: This patch series is currently being upstreamed to mainline Linux.
