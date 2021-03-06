README
^^^^^^

  This README discusses issues unique to NuttX configurations for the
  Atmel SAM4S Xplained  development board.  This board features the
  ATSAM4S16C MCU with 1MB FLASH and 128KB.

  The SAM4S Xplained features:

    - 120 MHz Cortex-M4 with MPU
    - 12MHz crystal (no 32.768KHz crystal)
    - Segger J-Link JTAG emulator on-board for program and debug
    - MICRO USB A/B connector for USB connectivity
    - IS66WV51216DBLL ISSI SRAM 8Mb 512K x 16 55ns PSRAM 2.5v-3.6v
    - Four Atmel QTouch buttons
    - External voltage input
    - Four LEDs, two controllable from software
    - Xplained expansion headers
    - Footprint for external serial Flash (not fitted)

Contents
^^^^^^^^

  - PIO Muxing
  - Development Environment
  - GNU Toolchain Options
  - IDEs
  - NuttX EABI "buildroot" Toolchain
  - NuttX OABI "buildroot" Toolchain
  - NXFLAT Toolchain
  - Buttons and LEDs
  - Serial Consoles
  - SAM4S Xplained-specific Configuration Options
  - Configurations

PIO Muxing
^^^^^^^^^^

  PA0   SMC_A17                  PB0   J2.3 default   PC0   SMC_D0
  PA1   SMC_A18                  PB1   J2.4           PC1   SMC_D1
  PA2   J3.7 default             PB2   J1.3 & J4.3    PC2   SMC_D2
  PA3   J1.1 & J4.1              PB3   J1.4 & J4.4    PC3   SMC_D3
  PA4   J1.2 & J4.2              PB4   JTAG           PC4   SMC_D4
  PA5   User_button BP2          PB5   JTAG           PC5   SMC_D5
  PA6   J3.7 optional            PB6   JTAG           PC6   SMC_D6
  PA7   CLK_32K                  PB7   JTAG           PC7   SMC_D7
  PA8   CLK_32K                  PB8   CLK_12M        PC8   SMC_NWE
  PA9   RX_UART0                 PB9   CLK_12M        PC9   Power on detect
  PA10  TX_UART0                 PB10  USB_DDM        PC10  User LED D9
  PA11  J3.2 default             PB11  USB_DDP        PC11  SMC_NRD
  PA12  MISO                     PB12  ERASE          PC12  J2.2
  PA13  MOSI                     PB13  J2.3 optional  PC13  J2.7
  PA14  SPCK                     PB14  N/A            PC14  SMC_NCS0
  PA15  J3.5                                          PC15  SMC_NSC1
  PA16  J3.6                                          PC16  N/A
  PA17  J2.5                                          PC17  User LED D10
  PA18  J3.4 & SMC_A14                                PC18  SMC_A0
  PA19  J3.4 optional & SMC_A15                       PC19  SMC_A1
  PA20  J3.1 & SMC_A16                                PC20  SMC_A2
  PA21  J2.6                                          PC21  SMC_A3
  PA22  J2.1                                          PC22  SMC_A4
  PA23  J3.3                                          PC23  SMC_A5
  PA24  TSLIDR_SL_SN                                  PC24  SMC_A6
  PA25  TSLIDR_SL_SNSK                                PC25  SMC_A7
  PA26  TSLIDR_SM_SNS                                 PC26  SMC_A8
  PA27  TSLIDR_SM_SNSK                                PC27  SMC_A9
  PA28  TSLIDR_SR_SNS                                 PC28  SMC_A10
  PA29  TSLIDR_SR_SNSK                                PC29  SMC_A11
  PA30  J4.5                                          PC30  SMC_A12
  PA31  J1.5                                          PC31  SMC_A13

Development Environment
^^^^^^^^^^^^^^^^^^^^^^^

  Either Linux or Cygwin on Windows can be used for the development environment.
  The source has been built only using the GNU toolchain (see below).  Other
  toolchains will likely cause problems. Testing was performed using the Cygwin
  environment.

GNU Toolchain Options
^^^^^^^^^^^^^^^^^^^^^

  The NuttX make system has been modified to support the following different
  toolchain options.

  1. The CodeSourcery GNU toolchain,
  2. The devkitARM GNU toolchain, ok
  4. The NuttX buildroot Toolchain (see below).

  All testing has been conducted using the NuttX buildroot toolchain.  However,
  the make system is setup to default to use the devkitARM toolchain.  To use
  the CodeSourcery, devkitARM or Raisonance GNU toolchain, you simply need to
  add one of the following configuration options to your .config (or defconfig)
  file:

    CONFIG_SAM34_CODESOURCERYW=y  : CodeSourcery under Windows
    CONFIG_SAM34_CODESOURCERYL=y  : CodeSourcery under Linux
    CONFIG_SAM34_DEVKITARM=y      : devkitARM under Windows
    CONFIG_SAM34_BUILDROOT=y      : NuttX buildroot under Linux or Cygwin (default)

  If you are not using CONFIG_SAM34_BUILDROOT, then you may also have to modify
  the PATH in the setenv.h file if your make cannot find the tools.

  NOTE: the CodeSourcery (for Windows), devkitARM, and Raisonance toolchains are
  Windows native toolchains.  The CodeSourcey (for Linux) and NuttX buildroot
  toolchains are Cygwin and/or Linux native toolchains. There are several limitations
  to using a Windows based toolchain in a Cygwin environment.  The three biggest are:

  1. The Windows toolchain cannot follow Cygwin paths.  Path conversions are
     performed automatically in the Cygwin makefiles using the 'cygpath' utility
     but you might easily find some new path problems.  If so, check out 'cygpath -w'

  2. Windows toolchains cannot follow Cygwin symbolic links.  Many symbolic links
     are used in Nuttx (e.g., include/arch).  The make system works around these
     problems for the Windows tools by copying directories instead of linking them.
     But this can also cause some confusion for you:  For example, you may edit
     a file in a "linked" directory and find that your changes had no effect.
     That is because you are building the copy of the file in the "fake" symbolic
     directory.  If you use a Windows toolchain, you should get in the habit of
     making like this:

       make clean_context all

     An alias in your .bashrc file might make that less painful.

  3. Dependencies are not made when using Windows versions of the GCC.  This is
     because the dependencies are generated using Windows pathes which do not
     work with the Cygwin make.

       MKDEP                = $(TOPDIR)/tools/mknulldeps.sh

  NOTE 1: The CodeSourcery toolchain (2009q1) does not work with default optimization
  level of -Os (See Make.defs).  It will work with -O0, -O1, or -O2, but not with
  -Os.

  NOTE 2: The devkitARM toolchain includes a version of MSYS make.  Make sure that
  the paths to Cygwin's /bin and /usr/bin directories appear BEFORE the devkitARM
  path or will get the wrong version of make.

IDEs
^^^^

  NuttX is built using command-line make.  It can be used with an IDE, but some
  effort will be required to create the project (There is a simple RIDE project
  in the RIDE subdirectory).
  
  Makefile Build
  --------------
  Under Eclipse, it is pretty easy to set up an "empty makefile project" and
  simply use the NuttX makefile to build the system.  That is almost for free
  under Linux.  Under Windows, you will need to set up the "Cygwin GCC" empty
  makefile project in order to work with Windows (Google for "Eclipse Cygwin" -
  there is a lot of help on the internet).

  Native Build
  ------------
  Here are a few tips before you start that effort:

  1) Select the toolchain that you will be using in your .config file
  2) Start the NuttX build at least one time from the Cygwin command line
     before trying to create your project.  This is necessary to create
     certain auto-generated files and directories that will be needed.
  3) Set up include pathes:  You will need include/, arch/arm/src/sam34,
     arch/arm/src/common, arch/arm/src/armv7-m, and sched/.
  4) All assembly files need to have the definition option -D __ASSEMBLY__
     on the command line.

  Startup files will probably cause you some headaches.  The NuttX startup file
  is arch/arm/src/sam34/sam_vectors.S.  You may need to build NuttX
  one time from the Cygwin command line in order to obtain the pre-built
  startup object needed by RIDE.

NuttX EABI "buildroot" Toolchain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  A GNU GCC-based toolchain is assumed.  The files */setenv.sh should
  be modified to point to the correct path to the Cortex-M3 GCC toolchain (if
  different from the default in your PATH variable).

  If you have no Cortex-M3 toolchain, one can be downloaded from the NuttX
  SourceForge download site (https://sourceforge.net/projects/nuttx/files/buildroot/).
  This GNU toolchain builds and executes in the Linux or Cygwin environment.

  1. You must have already configured Nuttx in <some-dir>/nuttx.

     cd tools
     ./configure.shsam4s-xplained/<sub-dir>

  2. Download the latest buildroot package into <some-dir>

  3. unpack the buildroot tarball.  The resulting directory may
     have versioning information on it like buildroot-x.y.z.  If so,
     rename <some-dir>/buildroot-x.y.z to <some-dir>/buildroot.

  4. cd <some-dir>/buildroot

  5. cp configs/cortexm3-eabi-defconfig-4.6.3 .config

  6. make oldconfig

  7. make

  8. Edit setenv.h, if necessary, so that the PATH variable includes
     the path to the newly built binaries.

  See the file configs/README.txt in the buildroot source tree.  That has more
  details PLUS some special instructions that you will need to follow if you are
  building a Cortex-M3 toolchain for Cygwin under Windows.

  NOTE:  Unfortunately, the 4.6.3 EABI toolchain is not compatible with the
  the NXFLAT tools.  See the top-level TODO file (under "Binary loaders") for
  more information about this problem. If you plan to use NXFLAT, please do not
  use the GCC 4.6.3 EABI toochain; instead use the GCC 4.3.3 OABI toolchain.
  See instructions below.

NuttX OABI "buildroot" Toolchain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  The older, OABI buildroot toolchain is also available.  To use the OABI
  toolchain:

  1. When building the buildroot toolchain, either (1) modify the cortexm3-eabi-defconfig-4.6.3
     configuration to use EABI (using 'make menuconfig'), or (2) use an exising OABI
     configuration such as cortexm3-defconfig-4.3.3

  2. Modify the Make.defs file to use the OABI conventions:

    +CROSSDEV = arm-nuttx-elf-
    +ARCHCPUFLAGS = -mtune=cortex-m3 -march=armv7-m -mfloat-abi=soft
    +NXFLATLDFLAGS2 = $(NXFLATLDFLAGS1) -T$(TOPDIR)/binfmt/libnxflat/gnu-nxflat-gotoff.ld -no-check-sections
    -CROSSDEV = arm-nuttx-eabi-
    -ARCHCPUFLAGS = -mcpu=cortex-m3 -mthumb -mfloat-abi=soft
    -NXFLATLDFLAGS2 = $(NXFLATLDFLAGS1) -T$(TOPDIR)/binfmt/libnxflat/gnu-nxflat-pcrel.ld -no-check-sections

NXFLAT Toolchain
^^^^^^^^^^^^^^^^

  If you are *not* using the NuttX buildroot toolchain and you want to use
  the NXFLAT tools, then you will still have to build a portion of the buildroot
  tools -- just the NXFLAT tools.  The buildroot with the NXFLAT tools can
  be downloaded from the NuttX SourceForge download site
  (https://sourceforge.net/projects/nuttx/files/).
 
  This GNU toolchain builds and executes in the Linux or Cygwin environment.

  1. You must have already configured Nuttx in <some-dir>/nuttx.

     cd tools
     ./configure.sh lpcxpresso-lpc1768/<sub-dir>

  2. Download the latest buildroot package into <some-dir>

  3. unpack the buildroot tarball.  The resulting directory may
     have versioning information on it like buildroot-x.y.z.  If so,
     rename <some-dir>/buildroot-x.y.z to <some-dir>/buildroot.

  4. cd <some-dir>/buildroot

  5. cp configs/cortexm3-defconfig-nxflat .config

  6. make oldconfig

  7. make

  8. Edit setenv.h, if necessary, so that the PATH variable includes
     the path to the newly builtNXFLAT binaries.

Buttons and LEDs
^^^^^^^^^^^^^^^^

  Buttons
  -------

  The SAM4S Xplained has two mechanical buttons. One button is the RESET button
  connected to the SAM4S reset line and the other is a generic user configurable
  button labeled BP2 and connected to GPIO PA5. When a button is pressed it
  will drive the I/O line to GND.
 
  LEDs
  ----

  There are four LEDs on board the SAM4X Xplained board, two of these can be
  controlled by software in the SAM4S:

      LED              GPIO
      ---------------- -----
      D9  Yellow LED   PC10
      D10 Yellow LED   PC17
 
  Both can be illuminated by driving the GPIO output to ground (low).

  These LEDs are not used by the board port unless CONFIG_ARCH_LEDS is
  defined.  In that case, the usage by the board port is defined in
  include/board.h and src/up_leds.c. The LEDs are used to encode OS-related
  events as follows:

    SYMBOL                Meaning                     LED state
                                                    D9       D10
    -------------------  -----------------------  -------- --------
    LED_STARTED          NuttX has been started     OFF      OFF
    LED_HEAPALLOCATE     Heap has been allocated    OFF      OFF
    LED_IRQSENABLED      Interrupts enabled         OFF      OFF
    LED_STACKCREATED     Idle stack created         ON       OFF
    LED_INIRQ            In an interrupt              No change
    LED_SIGNAL           In a signal handler          No change
    LED_ASSERTION        An assertion failed          No change
    LED_PANIC            The system has crashed     OFF      Blinking
    LED_IDLE             MCU is is sleep mode         Not used

  Thus if D9 is statically on, NuttX has successfully booted and is,
  apparently, running normmally.  If D10 is flashing at approximately
  2Hz, then a fatal error has been detected and the system has halted.

Serial Consoles
^^^^^^^^^^^^^^^

  USART0
  ------

  If you have a TTL to RS-232 convertor then this is the most convenient
  serial console to use.  UART1 is the default in all of these
  configurations.

    UART1 RXD  PB2   J1 pin 3   J4 pin 3
    UART1 TXD  PB3   J1 pin 4   J4 pin 4
    GND              J1 pin 9   J4 pin 9
    Vdd              J1 pin 10  J4 pin 10

  USART1 is another option:

    USART1 RXD PA21  J2 pin 6
    USART1 TXD PA22  J2 pin 1
    GND              J2 pin 9
    Vdd              J2 pin 10

  Yet another option is to use UART0 and the virtual COM port.  This
  option may be more convenient for long term development, but was
  painful to use during board bring-up.

  Virtual COM Port
  ----------------

  The SAM4S Xplained contains an Embedded Debugger (EDBG) that can be
  used to program and debug the ATSAM4S16C using Serial Wire Debug (SWD).
  The Embedded debugger also include a Virtual Com port interface over
  USART1.  Virtual COM port connections:

  AT91SAM4S16     ATSAM3U4CAU
  -------------- --------------
  PA9   RX_UART0  PA9_4S PA12
  PA10  TX_UART0  RX_3U  PA11

SAM4S Xplained-specific Configuration Options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

  CONFIG_ARCH - Identifies the arch/ subdirectory.  This should
  be set to:

    CONFIG_ARCH=arm

  CONFIG_ARCH_family - For use in C code:

    CONFIG_ARCH_ARM=y

  CONFIG_ARCH_architecture - For use in C code:

    CONFIG_ARCH_CORTEXM4=y

  CONFIG_ARCH_CHIP - Identifies the arch/*/chip subdirectory

    CONFIG_ARCH_CHIP="sam34"

  CONFIG_ARCH_CHIP_name - For use in C code to identify the exact
  chip:

    CONFIG_ARCH_CHIP_SAM34
    CONFIG_ARCH_CHIP_SAM4S
    CONFIG_ARCH_CHIP_ATSAM4S16C

  CONFIG_ARCH_BOARD - Identifies the configs subdirectory and
  hence, the board that supports the particular chip or SoC.

    CONFIG_ARCH_BOARD=sam4s-xplained (for the SAM4S Xplained development board)

  CONFIG_ARCH_BOARD_name - For use in C code

    CONFIG_ARCH_BOARD_SAM4S_XPLAINED=y

  CONFIG_ARCH_LOOPSPERMSEC - Must be calibrated for correct operation
  of delay loops

  CONFIG_ENDIAN_BIG - define if big endian (default is little
  endian)

  CONFIG_DRAM_SIZE - Describes the installed DRAM (SRAM in this case):

    CONFIG_DRAM_SIZE=0x00008000 (32Kb)

  CONFIG_DRAM_START - The start address of installed DRAM

    CONFIG_DRAM_START=0x20000000

  CONFIG_ARCH_IRQPRIO - The SAM3UF103Z supports interrupt prioritization

    CONFIG_ARCH_IRQPRIO=y

  CONFIG_ARCH_LEDS - Use LEDs to show state. Unique to boards that
  have LEDs

  CONFIG_ARCH_INTERRUPTSTACK - This architecture supports an interrupt
  stack. If defined, this symbol is the size of the interrupt
  stack in bytes.  If not defined, the user task stacks will be
  used during interrupt handling.

  CONFIG_ARCH_STACKDUMP - Do stack dumps after assertions

  CONFIG_ARCH_LEDS -  Use LEDs to show state. Unique to board architecture.

  CONFIG_ARCH_CALIBRATION - Enables some build in instrumentation that
  cause a 100 second delay during boot-up.  This 100 second delay
  serves no purpose other than it allows you to calibratre
  CONFIG_ARCH_LOOPSPERMSEC.  You simply use a stop watch to measure
  the 100 second delay then adjust CONFIG_ARCH_LOOPSPERMSEC until
  the delay actually is 100 seconds.

  Individual subsystems can be enabled:

    CONFIG_SAM34_RTC         - Real Time Clock
    CONFIG_SAM34_RTT         - Real Time Timer
    CONFIG_SAM34_WDT         - Watchdog Timer
    CONFIG_SAM34_UART0       - UART 0
    CONFIG_SAM34_UART1       - UART 1
    CONFIG_SAM34_SMC         - Static Memory Controller
    CONFIG_SAM34_USART0      - USART 0
    CONFIG_SAM34_USART1      - USART 1
    CONFIG_SAM34_HSMCI       - High Speed Multimedia Card Interface
    CONFIG_SAM34_TWI0        - Two-Wire Interface 0
    CONFIG_SAM34_TWI1        - Two-Wire Interface 1
    CONFIG_SAM34_SPI         - Serial Peripheral Interface
    CONFIG_SAM34_SSC         - Synchronous Serial Controller
    CONFIG_SAM34_TC0         - Timer Counter 0
    CONFIG_SAM34_TC1         - Timer Counter 1
    CONFIG_SAM34_TC2         - Timer Counter 2
    CONFIG_SAM34_TC3         - Timer Counter 3
    CONFIG_SAM34_TC4         - Timer Counter 4
    CONFIG_SAM34_TC5         - Timer Counter 5
    CONFIG_SAM34_ADC12B      - 12-bit Analog To Digital Converter
    CONFIG_SAM34_DACC        - Digital To Analog Converter
    CONFIG_SAM34_PWM         - Pulse Width Modulation
    CONFIG_SAM34_CRCCU       - CRC Calculation Unit
    CONFIG_SAM34_ACC         - Analog Comparator
    CONFIG_SAM34_UDP         - USB Device Port

  Some subsystems can be configured to operate in different ways. The drivers
  need to know how to configure the subsystem.

    CONFIG_GPIOA_IRQ
    CONFIG_GPIOB_IRQ
    CONFIG_GPIOC_IRQ
    CONFIG_USART0_ISUART
    CONFIG_USART1_ISUART
    CONFIG_USART2_ISUART
    CONFIG_USART3_ISUART

  ST91SAM4S specific device driver settings

    CONFIG_U[S]ARTn_SERIAL_CONSOLE - selects the USARTn (n=0,1,2,3) or UART
           m (m=4,5) for the console and ttys0 (default is the USART1).
    CONFIG_U[S]ARTn_RXBUFSIZE - Characters are buffered as received.
       This specific the size of the receive buffer
    CONFIG_U[S]ARTn_TXBUFSIZE - Characters are buffered before
       being sent.  This specific the size of the transmit buffer
    CONFIG_U[S]ARTn_BAUD - The configure BAUD of the UART.  Must be
    CONFIG_U[S]ARTn_BITS - The number of bits.  Must be either 7 or 8.
    CONFIG_U[S]ARTn_PARTIY - 0=no parity, 1=odd parity, 2=even parity
    CONFIG_U[S]ARTn_2STOP - Two stop bits

Configurations
^^^^^^^^^^^^^^

  Each SAM4S Xplained configuration is maintained in a sub-directory and
  can be selected as follow:

    cd tools
    ./configure.shsam4s-xplained/<subdir>
    cd -
    . ./setenv.sh

  Before sourcing the setenv.sh file above, you should examine it and perform
  edits as necessary so that BUILDROOT_BIN is the correct path to the directory
  than holds your toolchain binaries.

  And then build NuttX by simply typing the following.  At the conclusion of
  the make, the nuttx binary will reside in an ELF file called, simply, nuttx.

    make

  The <subdir> that is provided above as an argument to the tools/configure.sh
  must be is one of the following.

  NOTE:  These configurations use the mconf-based configuration tool.  To
  change any of these configurations using that tool, you should:

    a. Build and install the kconfig-mconf tool.  See nuttx/README.txt
       and misc/tools/

    b. Execute 'make menuconfig' in nuttx/ in order to start the
       reconfiguration process.

Configuration sub-directories
-----------------------------

  ostest:
    This configuration directory performs a simple OS test using
    examples/ostest.

    NOTES:

    1. This configuration provides test output on USART0 which is available
       on EXT1 or EXT4 (see the section "Serial Consoles" above).  The
       virtual COM port could be used, instead, by reconfiguring to use
       USART1 instead of USART0:

       System Type -> AT91SAM3/4 Peripheral Support
         CONFIG_SAM_USART0=y
         CONFIG_SAM_USART1=n

       Device Drivers -> Serial Driver Support -> Serial Console
         CONFIG_USART0_SERIAL_CONSOLE=y

       Device Drivers -> Serial Driver Support -> USART0 Configuration
         CONFIG_USART0_2STOP=0
         CONFIG_USART0_BAUD=115200
         CONFIG_USART0_BITS=8
         CONFIG_USART0_PARITY=0
         CONFIG_USART0_RXBUFSIZE=256
         CONFIG_USART0_TXBUFSIZE=256

    2. This configuration is set up to use the NuttX OABI toolchain (see
       above). Of course this can be reconfigured if you prefer a different
       toolchain.

  nsh:
    This configuration directory will built the NuttShell.

    NOTES:

    1. This configuration provides test output on UART1 which is available
       on J3 or J4 (see the section "Serial Consoles" above).  The
       virtual COM port could be used, instead, by reconfiguring to use
       UART0 instead of UART1:

       System Type -> AT91SAM3/4 Peripheral Support
         CONFIG_SAM_UART0=y
         CONFIG_SAM_UART1=n

       Device Drivers -> Serial Driver Support -> Serial Console
         CONFIG_UART0_SERIAL_CONSOLE=y

       Device Drivers -> Serial Driver Support -> USART0 Configuration
         CONFIG_UART0_2STOP=0
         CONFIG_UART0_BAUD=115200
         CONFIG_UART0_BITS=8
         CONFIG_UART0_PARITY=0
         CONFIG_UART0_RXBUFSIZE=256
         CONFIG_UART0_TXBUFSIZE=256

    2. This configuration is set up to use the NuttX OABI toolchain (see
       above). Of course this can be reconfigured if you prefer a different
       toolchain.

  nsh:
    This configuration directory will built the NuttShell.

    NOTES:

    1. This configuration provides test output on UART1 which is available
       on J3 or J4 (see the section "Serial Consoles" above).  The
       virtual COM port could be used, instead, by reconfiguring to use
       UART0 instead of UART1:

       System Type -> AT91SAM3/4 Peripheral Support
         CONFIG_SAM_UART0=y
         CONFIG_SAM_UART1=n

       Device Drivers -> Serial Driver Support -> Serial Console
         CONFIG_UART0_SERIAL_CONSOLE=y

       Device Drivers -> Serial Driver Support -> USART0 Configuration
         CONFIG_UART0_2STOP=0
         CONFIG_UART0_BAUD=115200
         CONFIG_UART0_BITS=8
         CONFIG_UART0_PARITY=0
         CONFIG_UART0_RXBUFSIZE=256
         CONFIG_UART0_TXBUFSIZE=256

    2. This configuration is set up to use the NuttX OABI toolchain (see
       above). Of course this can be reconfigured if you prefer a different
       toolchain.
