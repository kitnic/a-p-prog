# Programmer for 8-bit PIC devices built upon AVR (or Arduino)

For [Micro progmeter](https://github.com/jaromir-sukuba/micro_progmeter) project I want to deliver full set of open-source materials, but it would be shame if folks trying to replicate the project would have to buy another programmer to flash the PIC MCU, so I decided to do a little bit of brain stretching and implement PIC16F1xxx programmer with... Arduino. I really can't say I love this platform, but it is low cost and available.
Together with SDCC compiler this serves as completely open-source basis for many 8-bit PIC devices.

The current project status: Programmer working, sources need cleanup, perhaps rework of its structure (it grew out of its simple structure).
# List of supported devices is at the end of this page.


## Hardware
There are two options fo hardware for this project:

### Hardware option 1 - Arduino hardware
Take any Arduino with ATmega328P, like Uno or most of cheap chinese knock-off boards. Considering the target has its own power supply, connect GND, MCLR, PGC and PGD lines to respective pins on arduino as follows:

| Arduino pin | AVR pin | Target PIC pin | Comment                                                                     |
|-------------|---------|----------------|-----------------------------------------------------------------------------|
| GND         | GND     | GND            | common ground                                                               |
| 5V          | VDD     | VDD            | optional*- power supply for PIC MCU, you may power target from other source |
| A3          | PC3     | MCLR           | reset line of PIC MCU                                                       |
| A1          | PC1     | PGD            | programming data line                                                       |
| A0          | PC0     | PGC            | programming clock line                                                      |

* If you are powering target PIC from other power source (in circuit programming), series resistors (something like 470-1000 Ohms) in MCLR, PGC and PGD lines are good idea, it will somehow decrease currents flowing through IO pins due to power supplies differences between arduino and target board. In this configuration, even targets running at 3,3V are possible to program, though it is dirty practice.

Power it up, no smoke should be released. Run arduino IDE, open programmer firmware from here (/fw/pp.ino), complie and upload to arduino board - now you have PIC programmer ready to go. Go to software below.

The programmer is proven to work with some generic Uno board from china, as well as chinese arduino-nano clone
![img_5494](https://cloud.githubusercontent.com/assets/6984904/17290864/ddd9d386-57e0-11e6-8d15-55c6c0e015a6.JPG)

### Hardware option 2 - dedicated board
I designed this little board, see at /hw directory
![img_4319](https://cloud.githubusercontent.com/assets/6984904/17289293/5cedd6c0-57d9-11e6-86b8-8d692eaa24e3.JPG)
Considering the target has its own power supply, connect GND, MCLR, PGC and PGD lines to respective pins on PIC programmer (notice the pinout similar to PICkit programmers). Vdd line is not currently used ad it is not needed for operation of programmer, but future revisions of firmware may take advantage of this pin and detect target VDD.
The hardware works with both FT232RL and CY7C65213 in place of USB/serial converter. Both were proven to work, with no other hardware difference.
![img_4320](https://cloud.githubusercontent.com/assets/6984904/17289353/a3ccf6d4-57d9-11e6-9d4f-595633e7841a.JPG)

Use [another arduino](https://www.arduino.cc/en/Tutorial/ArduinoISP) (or proper ISP programmer) to load Arduino UNO bootloader to PIC programmer board (performed only once), turning it into regular arduino compatible board.

![img_4329](https://cloud.githubusercontent.com/assets/6984904/17289342/98207cf2-57d9-11e6-9b62-caba0b140ca5.JPG)

Ensure JP2 is closed, then you can load new firmware into PIC programmer using regular Arduino IDE. Open jumper JP2. Now you have your programmer ready to go, move on to software.
The firmware should be able to compile outside Arduino IDE as it doesn't contain any Arduino-specific stuff, though I haven't tried that.



## Software

When running under Linux, download source from this repository and run

`gcc -Wall pp3.c -o pp3`

This should build executable pp3. When working under windows, you can download compiled binary from this repository. Alternatively, you can build it from source - install minGW and run

`gcc -Wall pp3.c -o pp3`

ie. the same procedure as on Linux. This should result in silent build with pp3.exe executable created.

Running the executable with no parameters should only bring banner "pp programmer". Though running with basic set of parameters

`./pp3 -c /dev/ttyACM0 -t 16f1829 file.hex`

under Linux, where -c parameter denotes port to be accessed, -t parameter defines PIC to be programmed and last parameter is hex file to be downloaded; or

`pp3.exe -c COM30 -t 16f1829 file.hex`

under Windows should program the target PIC; with expected result:

	$ ./pp3 -c /dev/ttyACM0 -t 16f1829 file.hex
    Opening serial port
    Device ID 0x27E4
    Programming FLASH (16384 B in 256 pages)
    Programming config
    Verifying FLASH (16384 B in 256 pages)
    Verifying config
    Releasing MCLR


## Notes on software

If you are running the hardware on generic arduino board or you forget to open jumper JP2 after loading firmware on dedicated hardware, you may need to insert waiting time after opening serial port and before communication - to ensure Arduino bootloader times out after opening serial port and takes control to programmer firmware. It should look like this

`pp3.exe -c COM30 -s 1700 -t 16f1829 file.bin`

where number after -s switch defines the number of miliseconds to wait after opening the serial port.

You may omit the actual programming using -p switch or verification using -n switch, when using both the programmer only checks target device signature and exits.

`$ ./pp3 -c /dev/ttyACM0 -p -n -t 16f1829 file.bin`

    Opening serial port
    Device ID 0x27E4
    Releasing MCLR

you can add some debug output info using -v parameter, ranging from -v 1 to -v 4. It may be suitable for debugging, -v 4 prints out all byte transaction on serial port, so be prepared for huge output.


## Supported devices

Obviously, there is more supported than verified (tested) devices. I tried to test at least one device from specific family. The members from one family are usually very very similar, giving me enough confidence to mark them as supproted. Of course, typos can happen, but those are easy to correct.

|              |        | 
|--------------|--------| 
| DEVICE       | TESTED | 
| PIC12F1501   | YES    | 
| PIC12F1571   |        | 
| PIC12F1572   | YES    | 
| PIC12F1612   | YES    | 
| PIC12F1822   | YES    | 
| PIC12F1840   | YES    | 
| PIC12LF1501  |        | 
| PIC12LF1571  |        | 
| PIC12LF1572  |        | 
| PIC12LF1612  |        | 
| PIC12LF1822  |        | 
| PIC12LF1840  |        | 
| PIC16F1454   |        | 
| PIC16F1455   | YES    | 
| PIC16F1459   |        | 
| PIC16F1503   | YES    | 
| PIC16F1507   | YES    | 
| PIC16F1508   |        | 
| PIC16F1509   | YES    | 
| PIC16F1512   |        | 
| PIC16F1513   |        | 
| PIC16F1516   | YES    | 
| PIC16F1517   |        | 
| PIC16F1518   |        | 
| PIC16F1519   |        | 
| PIC16F1528   |        | 
| PIC16F1529   |        | 
| PIC16F1574   |        | 
| PIC16F1575   | YES    | 
| PIC16F1578   |        | 
| PIC16F1579   |        | 
| PIC16F1613   |        | 
| PIC16F1614   |        | 
| PIC16F1615   | YES    | 
| PIC16F1618   |        | 
| PIC16F1619   |        | 
| PIC16F1703   | YES    | 
| PIC16F1704   |        | 
| PIC16F1705   |        | 
| PIC16F1707   |        | 
| PIC16F1708   |        | 
| PIC16F1709   |        | 
| PIC16F1713   |        | 
| PIC16F1716   |        | 
| PIC16F1717   |        | 
| PIC16F1718   | YES    | 
| PIC16F1719   |        | 
| PIC16F1764   |        | 
| PIC16F1765   | YES    | 
| PIC16F1768   |        | 
| PIC16F1769   |        | 
| PIC16F1782   |        | 
| PIC16F1783   | YES    | 
| PIC16F1784   |        | 
| PIC16F1786   |        | 
| PIC16F1787   |        | 
| PIC16F1788   |        | 
| PIC16F1789   |        | 
| PIC16F1824   |        | 
| PIC16F1825   |        | 
| PIC16F1826   |        | 
| PIC16F1827   |        | 
| PIC16F1828   |        | 
| PIC16F1829   | YES    | 
| PIC16F1847   |        | 
| PIC16F1933   |        | 
| PIC16F1934   |        | 
| PIC16F1936   |        | 
| PIC16F1937   |        | 
| PIC16F1938   |        | 
| PIC16F1939   |        | 
| PIC16F1946   |        | 
| PIC16F1947   |        | 
| PIC16LF1454  |        | 
| PIC16LF1455  |        | 
| PIC16LF1459  |        | 
| PIC16LF1503  | YES    | 
| PIC16LF1507  |        | 
| PIC16LF1508  |        | 
| PIC16LF1509  |        | 
| PIC16LF1512  |        | 
| PIC16LF1513  |        | 
| PIC16LF1516  |        | 
| PIC16LF1517  |        | 
| PIC16LF1518  |        | 
| PIC16LF1519  |        | 
| PIC16LF1528  |        | 
| PIC16LF1529  |        | 
| PIC16LF1574  |        | 
| PIC16LF1575  |        | 
| PIC16LF1578  |        | 
| PIC16LF1579  |        | 
| PIC16LF1613  |        | 
| PIC16LF1614  |        | 
| PIC16LF1615  |        | 
| PIC16LF1618  |        | 
| PIC16LF1619  |        | 
| PIC16LF1703  |        | 
| PIC16LF1704  |        | 
| PIC16LF1705  |        | 
| PIC16LF1707  |        | 
| PIC16LF1708  |        | 
| PIC16LF1709  |        | 
| PIC16LF1713  |        | 
| PIC16LF1716  |        | 
| PIC16LF1717  |        | 
| PIC16LF1718  |        | 
| PIC16LF1719  |        | 
| PIC16LF1764  |        | 
| PIC16LF1765  |        | 
| PIC16LF1768  |        | 
| PIC16LF1769  |        | 
| PIC16LF1782  |        | 
| PIC16LF1783  |        | 
| PIC16LF1784  |        | 
| PIC16LF1786  |        | 
| PIC16LF1787  |        | 
| PIC16LF1788  |        | 
| PIC16LF1789  |        | 
| PIC16LF1824  |        | 
| PIC16LF1825  |        | 
| PIC16LF1826  |        | 
| PIC16LF1827  |        | 
| PIC16LF1828  |        | 
| PIC16LF1829  | YES    | 
| PIC16LF1847  |        | 
| PIC16LF1902  |        | 
| PIC16LF1903  | YES    | 
| PIC16LF1904  |        | 
| PIC16LF1906  |        | 
| PIC16LF1907  |        | 
| PIC16LF1933  |        | 
| PIC16LF1934  |        | 
| PIC16LF1936  |        | 
| PIC16LF1937  |        | 
| PIC16LF1938  |        | 
| PIC16LF1939  |        | 
| PIC16LF1946  |        | 
| PIC16LF1947  |        | 
| PIC18F23K22  |        | 
| PIC18F24J10  |        | 
| PIC18F24J11  |        | 
| PIC18F24J50  | YES    | 
| PIC18F24K22  | YES    | 
| PIC18F25J10  |        | 
| PIC18F25J11  |        | 
| PIC18F25J50  |        | 
| PIC18F25K22  |        | 
| PIC18F25K50  | YES    | 
| PIC18F26J11  | YES    | 
| PIC18F26J13  |        | 
| PIC18F26J50  | YES    | 
| PIC18F26J53  |        | 
| PIC18F26K22  | YES    | 
| PIC18F27J13  |        | 
| PIC18F27J53  | YES    | 
| PIC18F43K22  |        | 
| PIC18F44J10  |        | 
| PIC18F44J11  |        | 
| PIC18F44J50  |        | 
| PIC18F44K22  |        | 
| PIC18F45J10  |        | 
| PIC18F45J11  |        | 
| PIC18F45J50  |        | 
| PIC18F45K22  |        | 
| PIC18F46J11  |        | 
| PIC18F46J13  |        | 
| PIC18F46J50  |        | 
| PIC18F46J53  |        | 
| PIC18F46K22  |        | 
| PIC18F47J13  |        | 
| PIC18F47J53  |        | 
| PIC18LF23K22 |        | 
| PIC18LF24J10 |        | 
| PIC18LF24J11 |        | 
| PIC18LF24J50 |        | 
| PIC18LF24K22 |        | 
| PIC18LF25J10 |        | 
| PIC18LF25J11 |        | 
| PIC18LF25J50 |        | 
| PIC18LF25K22 |        | 
| PIC18LF25K50 | YES    | 
| PIC18LF26J11 |        | 
| PIC18LF26J13 |        | 
| PIC18LF26J50 |        | 
| PIC18LF26J53 |        | 
| PIC18LF26K22 |        | 
| PIC18LF27J13 |        | 
| PIC18LF27J53 |        | 
| PIC18LF43K22 |        | 
| PIC18LF44J10 |        | 
| PIC18LF44J11 |        | 
| PIC18LF44J50 |        | 
| PIC18LF44K22 |        | 
| PIC18LF45J10 |        | 
| PIC18LF45J11 |        | 
| PIC18LF45J50 |        | 
| PIC18LF45K22 |        | 
| PIC18LF46J11 |        | 
| PIC18LF46J13 |        | 
| PIC18LF46J50 |        | 
| PIC18LF46J53 |        | 
| PIC18LF46K22 |        | 
| PIC18LF47J13 |        | 
| PIC18LF47J53 |        | 






The whole project is licensed under MIT license, see LICENSE.md file.
Some more details to be found here https://hackaday.io/project/8559-pic16f1xxx-arduino-based-programmer
