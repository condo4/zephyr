.. _knx_knxboard:

KnxBoard reference hardware
##############################

`KnxBoard <https://github.com/condo4/knxboard>`_ is an openly published
reference board for this stack, pairing an STM32G4 with the NCN5130
transceiver documented in :ref:`knx_ncn5130`. Its schematics, PCB and full
board documentation live in that repository; the board support files
themselves (devicetree, Kconfig, ``board.cmake``) are in-tree at
:zephyr_file:`boards/kazoe/knxboardv4` and :zephyr_file:`boards/kazoe/knxboardv5`.

Carrier board and daughterboards
***********************************

KnxBoard is deliberately generic — on its own it only bridges an STM32 to
the KNX TP1 bus, in the spirit of an Arduino base board. It exposes a
mating header so a **daughterboard** can add whatever sensors or actuators
a particular product needs, each with its own :ref:`Zephyr application
<knx_application>` and board overlay, while sharing the same carrier board
devicetree and drivers.

Two daughterboards demonstrate the pattern in this repository:

* a **DS18B20 1-Wire** temperature probe board, using the classic
  UART/1-Wire bus-master wiring (Maxim AN214) on a free USART;
* a **PT100** temperature board, sampling 4 channels. Its first revision
  read the sensors through the carrier's own ADC, which ties the measured
  circuit's ground to the STM32 (and hence the KNX bus) ground — workable,
  but not ideal for a sensor board isolated from the installation it
  measures. A later revision moved the ADC onto the daughterboard itself,
  behind an I2C isolator and an isolated DC/DC supply, so the measurement
  ground floats free of the STM32/KNX-bus ground.

Hardware summary
*******************

.. list-table::
   :header-rows: 1

   * -
     - KnxBoard V5 (current)
     - KnxBoard V4
   * - MCU
     - STM32G491KEU6, UFQFPN32 — 512 KB flash, 112 KB RAM, 128 MHz PLL
     - same
   * - HSE
     - 16 MHz from the NCN5130 ``XCLK`` output (``hse-bypass``)
     - same
   * - KNX transceiver UART
     - LPUART1 on PA2/PA3, 9-bit @ 38400 bps, DMA async
     - USART1 on PA9/PA10
   * - Debug console
     - USART2, TX PB3 / RX PA15, 115200 bps
     - USART2 on PA2/PA3, 115200 bps
   * - KNX programming LED / button
     - PF1 / PA0
     - PB0 / PA8
   * - NCN5130 ``RESETB``
     - wired to the STM32's ``NRST``
     - —
   * - NVS storage
     - 4 KB, last flash page
     - same

Two wiring details that firmware relying on this board must respect:

* The NCN5130's ``XCLK`` output is the STM32's *only* clock source, so
  Analog Control Register 0's ``XCLKEN`` bit must stay enabled.
* The NCN5130's ``VDD2`` feeds the board's 12 V rail, so ``DC2EN`` must
  stay enabled too.

Known hardware/devicetree gaps
*********************************

* On V5, the NCN5130's ``SAVEB`` brownout-warning signal is wired (to PA1)
  but not yet declared in the devicetree, so no firmware currently uses it
  — a KNX end device is expected to save its non-volatile state on a bus
  brownout (the KNX ``Usave`` signal), which this omission currently
  leaves unimplemented.
* ``ANAOUT`` (bus-voltage monitor, needed for the mandatory
  ``A_ADC_Read`` channel 1) is unconnected on V5.

See the `KnxBoard <https://github.com/condo4/knxboard>`_ repository for the
full schematic, PCB, and the board's own change history across revisions.
