# Board and module tests

## test_leds

Test purpose: to test that both status LEDs work

Files needed: ```test_leds.py```

Test procedure: 
```python
from test_pins import *
test()
## Red and Green LED's should alternately flash 4 times
```

## test_pins

Test purpose: To test that all output pins work and are wired correctly.

Files needed: ```test_pins.py```

Test procedure:
```python
from test_pins import *
list()  # show pin mappings
flash(4)  # flash GP4
test()  # interactive tester
# choose the pin number, press RETURN many times to re-test same pin

```
## test_sdcard_spi

Test purpose: To test that the SD card adaptor is correctly wired

Files needed: ```test_sdcard_spi.py```

Other parts needed: SDCard breakout board, working micro-SDCard

Test procedure:
```python
# wire up SDCard (GP10=SCK, GP11=MOSI, GP12=MISO, GP13=CS, power and GND
from test_sdcard_spi import *
test()  # expect PASS
```

## test_radio_spi

Test purpose: To test that the radio is wired correctly and responds

Files needed: ```test_radio_spi.py```

Other parts needed: RFM69HCW Adafruit breakout

Test procedure:
```python
# wire up Radio (GP0=G0, GP1=CS, GP2=SCK, GP3=MOSI, GP4=MISO, GP6=RES, GP7=EN)
from test_radio_spi import *
test()  # expect PASS (radiover 0x10=0x24)
```


## test_radio

Test purpose: To test radio transmit/receive and range

Files needed: ```test_radio.py, radio.py, myboard.py, platdeps.py, dttk.py```

Other parts needed: RFM69HCW Adafruit breakout (passes test_radio_spi first)

Test Procedure:
```python
# build two boards with radios
# sender board needs GP16 pulled to 3V3
# receiver board needs GP16 pulled to ground

# on both boards, load the code:
from test_radio import *

# then run the tester
test_radio()
```

On the sender, you should see it send repeated packets of 66 bytes as follows:
```python
test_radio()
BAUD_RATE:250000 (0081)
test_send()
tx[1]: 250000 66
tx[2]: 250000 66
tx[3]: 250000 66
```
On the receiver, you should see it receive repeated packets and decode them 
correctly as follows:

```python
>>> test_radio()
BAUD_RATE:250000 (0081)
test_recv()
rx[1]: BR:250000, RSSI:-81.0, len:66
hex: 41 30 31 32 33 34 35 36 37 38 39 30 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 
4E 4F 50 51 52 53 54 55 56 57 58 59 5A 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 
6F 70 71 72 73 74 75 76 77 78 79 7A 40 2A
txt: A 0 1 2 3 4 5 6 7 8 9 0 A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 
a b c d e f g h i j k l m n o p q r s t u v w x y z @ *
rx[2]: BR:250000, RSSI:-82.0, len:66
hex: 41 30 31 32 33 34 35 36 37 38 39 30 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 
4E 4F 50 51 52 53 54 55 56 57 58 59 5A 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 
6F 70 71 72 73 74 75 76 77 78 79 7A 40 2A
txt: A 0 1 2 3 4 5 6 7 8 9 0 A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 
a b c d e f g h i j k l m n o p q r s t u v w x y z @ *
```

You can test the range, as you get to the end of the workable range, data
will get corrupted or not be received.


## sdtool

Test purpose: Test that you can read and write blocks to the SDCard

Files needed: ```sdtool.py, sdcard.py```

Other parts needed: Adafruit SDCard breakout, SDCard, working test_sdcard_spi

Test procedure:

```python
# wire card up as: GP10=SCK, GP11=MOSI, GP12=MISO, GP13=SCK, power and GND
# format as MSDOS-FAT in a PC drive first, give the drive a name

import sdtool

# i (info)
# m (show master boot record)
# p (dump partition table)
# b 0 0 (dump partition 0 block 0 - should see your label)
# W (perform write tests)
```

For more detailed instructions about ```sdtool``` see:
[sdtool_instructions](sdtool_instructions.md)



# File Transfer Tests

For these tests, you might have multiple links configured. If this is the case,
when you import the ftag module, you might get this message, in which case,
choose the U option for the UART link (as radio.py isn't released yet).

```python
>>> import ftag
OPTION=1
[U]ART or [R]adio [UR]? U
>>> 
```

## UART loopback test

Test Purpose: test that the file transfer works over a UART on same device

Files Needed:
```python
platdeps.py
myboard.py
dttk.py
ftag.py
ftag_pico.py
perf.py
tasking.py
radio.py
test35k.jpg
testdata.txt
```

Test Procedure:

Short pins 1 and 2 on Pico (UART0_TX -> UART0_RX)

```bash
# use Thonny, or putty, or on mac/linux, use 'screen' command e.g.
$ screen /dev/tty.usbmodem14211
```

```python
import ftag
ftag.help()
ftag.loopback()
```

## UART transfer between two Pico's

Test Purpose: Test that you can transfer a file over a wire between two Pico devices

Files Needed:
```python
platdeps.py
myboard.py
dttk.py
ftag.py
ftag_pico.py
perf.py
tasking.py
radio.py
test35k.jpg
testdata.txt
```

Test Procedure:

Connect PicoA(sender) Pin1 to PicoB(receiver) Pin2

Connect PicoA(sender) GND-Pin38 to PicoB(receiver) GND-Pin38

NOTE: best to start the receiver, before you start the sender.
However, the protocol is resilient, and if you get to the END record
before all blocks are received, the receiver will prompt you to re-run
send() to get the remaining blocks.

```python
# use Thonny, or putty, or on mac/linux, use 'screen' command
# on Pico B (receiver)
import ftag
  OPTION=0
  [U]ART or [R]adio [UR]? U
ftag.receive()
```

```python
# use Thonny, or putty, or on mac/linux, use 'screen' command
# on Pico A (sender)
import ftag
  OPTION=1
  [U]ART or [R]adio [UR]? U
ftag.send()
```

## Radio transfer beween two Pico devices

Test Purpose: Transfer a JPG image wirelessly between two pico devices

Files Needed:
```python
platdeps.py
myboard.py
dttk.py
ftag.py
ftag_pico.py
perf.py
tasking.py
radio.py
test35k.jpg
testdata.txt
```

Test Procedure:

When first importing ftag from boot, choose the R option on both Pico's
and make sure the radio has previously passed the test_spi_radio and
test_radio tests.

Run a file transfer at both ends, like this:

Start the receiver first...

```python
# use Thonny, or putty, or on mac/linux, use 'screen' command
# on Pico B (receiver)
import ftag
  OPTION=0
  [U]ART or [R]adio [UR]? R
ftag.receive()
```

Then start the sender...

```python
# use Thonny, or putty, or on mac/linux, use 'screen' command
# on Pico A (sender)
import ftag
  OPTION=1
  [U]ART or [R]adio [UR]? R
ftag.send()
```

The transfer takes a little more than a minute in normal circumstances.
When the sender has finished the transfer, you should now see a file on
the receiver pico filing system called 'received.jpg'.

```
finished_ok:files identical:received.jpg                                       
STATS:rx
link: tot:2726 badlen:0 shorthdr:1 long:0 seqno:125 crc:10
xfer: T:77 blk:695 by:34710 PPS:9 BPS:450
receive complete
```

```python
import os
os.listdir()
["received.jpg", ...]
```

NOTE: If the transfer receives too many damaged packets, the sender will
stop but the receiver will stay waiting - at this point if you just
re-start the sender again, the receiver will re-sync and collect any
packets that it missed the first time round.

NOTE 2: The file transfer currently stores the file in RAM, so don't
power off your receiver or stop the program, and you will then be able
to use the resume feature at the sender to get the missing blocks
transferred.

Use whatever method works for you, to transfer this received.jpg from the 
receiver Pico onto your host computer. Use an image viewing program
on your host computer to verify that you have received the image intact.


# Tips and tricks

## TIP: loading code to both Picos

If you are able to install code on your machine, this is a faster way to
get all files onto both Pico devices:

Install the rshell utility

```bash
pip3 install rshell
```

Connect both Picos via USB to your computer, then use the provided script to
copy all the right files onto (all) connected Pico devices.

```bash
./load_pico
```

## TIP: Load the transferred image onto your host PC (using rshell)

```bash
rshell
cp /pyboard/received.jpg .
exit

open received.jpg  # should open and display the image on host PC
```

