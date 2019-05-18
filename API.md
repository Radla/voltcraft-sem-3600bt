
## A. Characteristics

## Initialisation and authorisation  

```
Write Request Handle 0x018

03 e3 07 05 0a 14 1d 26 53 03
|  |  |  |  |  |  |  |  |  +  Byte 9: Secret (byte 1)
|  |  |  |  |  |  |  |  + Byte 8: Secret key (byte 2)
|  |  |  |  |  |  |  + Byte 7: Seconds in hex 
|  |  |  |  |  |  + Byte 6: Minute in hex
|  |  |  |  |  + Byte 5: Hour of day in hex   
|  |  |  |  + Byte 4: Day in month in hex
|  |  |  + Byte 3: Month in hex, where 01 is January and 0C is December
|  |  + Byte 2: Year in hex (byte 1)
|  + Byte 1: Year in hex (byte 2)
+ Byte 0: Command, "03" for setting time
```

## Enable / Disable command notifications
```
Write Request Handle 0x019

01 00
|  + always "00"
+ 01=Enable, 00=disable
```

## Power
### Turn power on
```
Write Request Handle 0x018
04 01
```

### Turn power off
```
Write Request Handle 0x018
04 00
```

## Schedulers
### Request scheduler setting
```
Write Request 0x0018

0e 00 00 05
|  |  |  + always "05"
|  |  + always "00"
|  + ID of timer to query, 00 - 05
+ Command, "0e" for quering time
```

### Interprete scheduler notification

```
Notification handle = 0x0018
value:
0e 00 00 82 81 02 03 04
0e id 00 on ah mm hh mm
|  |  |  |  |  |  |  + End minutes
|  |  |  |  |  |  + Bit 8: Action (1=turn on, 0=turn off), End hours
|  |  |  |  |  + Start minutes
|  |  |  |  + Bit 8: Action (1=turn on, 0=turn off), Start hours
|  |  |  + Activate: Bit 1: Sun 
                     Bit 2: Mon 
                     Bit 3: Tue 
                     Bit 4: Wed 
                     Bit 5: Thu
                     Bit 6: Fri
                     Bit 7: Sat
                     Bit 8: Active (1=active, 0=inactive)
|  |  + always "00"
|  + ID of scheduler, 0 - 5
+ "0c" notification for scheduler
```

### Set scheduler
```
Write Request 0x0018
0c id 00 on ah mm hh mm
|  |  |  |  |  |  |  + End minutes
|  |  |  |  |  |  + Action (bit 8) and End hours
|  |  |  |  |  + Start minutes
|  |  |  |  + Action (bit 8) and start hours
|  |  |  + Activate: off = "00", Bits 0 - 7: week mask, Bit 8: Action
|  |  + always "00"
|  + ID of scheduler, 0 - 5
+ "0c" command for scheduler
```

ah - action and hours in hex, hours 0 - 23
     turn on: 128 + hh
     turn off: hh

### Reset scheduler

```
Write Request 0x0018
0c id 00 00 00 00 00 00
```

## Countdown

### Interprete countdown notification

```
Notification handle = 0x0018
value: 
06 93 0b 04
06 ah mm ss

ah > 128: Turn on action, hh = ah - 128
ah < 128: Turn off action , hh = ah
```

### Set countdown

```
Write Request 0x0018
06 ah mm

ah - action and hours in hex, hours 0 - 23
     turn on: 128 + hh
     turn off: hh
mm - minutes in hex, 0 - 59 
```

## Reset countdown

```
Write Request 0x0018
06 00 00
```

## Overload

### Request overload setting

```
Write Request Handle 0x018
16
```

### Interprete overload notification

```
Notification handle = 0x0018
value:
16 80 e8 03
16 aa ww ww
        + Watt (high byte)
      + Watt (low byte)
   + Status: Active + Power off + Buzzer
```


### Set overload

```
Write Request 0x0018
15 aa ww ww
        + Watt (high byte)
      + Watt (low byte)
   + Status: Active + Power off + Buzzer
```

### Reset overload

```
Write Request 0x0018
15 00 00 00
```


## Standby 

### Request standby settings

```
Write Request 0x0018
14

Notification handle = 0x0018 value: 14 80 48 0d
```

### Interprete standby notification

```
Notification handle = 0x0018 
value:
14 80 48 0d
14 00 00 00
|  |  |  + low mark, Byte 1 in hex, here 3.4 Watt
|  |  + Watt low mark, Byte 2 in hex, here 3.4 Watt
|  + Action, 80 = turn off
```

### Set standby

```
Write Request 0x0018
13 aa ww ww
13 00 00 00
|  |  |  + low mark, Byte 1 in hex, here 3.4 Watt
|  |  + Watt low mark, Byte 2 in hex, here 3.4 Watt
|  + Action, 80 = turn off
```

### Reset standby

```
Write Request 0x0018
13 00 00 00
```

## Realtime measurement

## Enable / Disable realtime measurement notifications
```
Write Request Handle 0x013

01 00
|  + always "00"
+ 01=Enable, 00=disable
```

### Unubscribe from measurements 
```
Write Request 0x0013
0000
```

### Interprete realtime measurement notification

Characteristic: 0000fee1-494c-4f47-4943-544543480000

```
Notification handle = 0x0012
value:
01 03 23 85 01 00 34 01 42 77 01 05 18 02 49 97
ss uv vv vv ua aa aa uw ww ww up pp pp uf ff ff

ss - State
0 = off
1 = on
2 = countdown

uv, ua, uw, up, uf define count of digits before comma

1 -> 0.000
2 -> 00.00
3 -> 000.0
4 -> 00000
5 -> 0.000
other: 0.0

v - Voltage
a - Ampere
w - Watts
p - Power factor
f - Frequency

All values are just in decimal
```

## Stored measurements

### Request interval of stored measurement

You can't request an interval longer than 8 hours.

89 Days back possible!

```
01 00 00 05
|  |  |  + Offset in hours from Start (range from 1 to 8)
|  |  | "00" for today
|  + Begin interval in steps of 8 hours, 0=0h-7h, 8=8h-15h, 16=16-23h
+ Command, "01" for quering data

Example:
01 00 00 01
-> queries interval from 00:00 to 01:00

01 00 00 08
-> queries interval from 00:00 to 08:00

01 08 00 01
-> queries interval from 08:00 to 09:00

01 08 00 05
-> queries interval from 08:00 to 13:00

01 08 00 08
-> queries interval from 08:00 to 16:00

01 0F 00 04
-> queries interval from 16:00 to 20:00

01 0F 00 08
-> queries interval from 16:00 to 24:00

```

### Interprete stored measerment notification

_TODO_

### Reset stored data 

Characteristic: 0000fee3-494c-4f47-4943-544543480000

```
Write request 0x0018
19 -> clear power-on time
17 -> ???? delete stored data
18 -> request power-on time -> notification
```

## Device masterdata

### Device Information


```
char-read-uuid 00002a00-0000-1000-8000-00805f9b34fb
handle: 0x0003   value: 57 69 54 20 50 6f 77 65 72 20 4d 65 74 65 72
=> WiT Power Meter

char-read-uuid 00002a25-0000-1000-8000-00805f9b34fb
handle: 0x0020   value: 53 4e 3a 20 30 30 30 30 30 30 00
=> SN: 000000

char-read-uuid 00002a26-0000-1000-8000-00805f9b34fb
handle: 0x0022   value: 46 2f 57 3a 20 56 30 31 2e 33 32 00
=> F/W: V01.32

char-read-uuid 00002a27-0000-1000-8000-00805f9b34fb
handle: 0x0024   value: 48 2f 57 3a 20 56 30 30 2e 30 30 00
=> H/W: V00.00

char-read-uuid 00002a28-0000-1000-8000-00805f9b34fb
handle: 0x0026   value: 53 2f 57 3a 20 56 30 31 2e 31 33 00
=> S/W: V01.13

char-read-uuid 00002a29-0000-1000-8000-00805f9b34fb
handle: 0x0028   value: 57 69 74 74 65 63 68 20 43 6f 6d 70 61 6e 79 20 4c 74 64
=> Wittech Company Ltd
```

## ???

```
char-write-req 18 17
Notification handle = 0x0018 value: 17 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
                                    1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17
```

## Power-on time


### Request power-on time 
```
char-write-req 18 18
Notification handle = 0x0018 value: 18 01 00 00
                                       mm
```

### Reset power-on time 
```
char-write-req 18 19
```
