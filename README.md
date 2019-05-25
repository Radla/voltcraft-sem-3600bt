# voltcraft-sem-3600bt
_"A full-features shell script in order to manage Voltcrafts's bluetooth smart energy meter, switch and scheduler with Linux and Raspberry Pi"_

The Voltcraft SEM-3600BT is a remote 230V switch and smart energy meter. It was sold by Conrad Elektronic in Germany a few years ago. For details take a look at [Amazon's](https://www.amazon.de/Voltcraft-SEM-3600BT-Energiekosten-Messger%C3%A4t-Bluetooth%C2%AE-Schnittstelle-Darstellung/dp/B00IWS7V8G). In other countries it is probably known as "Wittech WiTenergy E100".

In comparison to many remote switches which use the 433MHz band the SEM-3600BT is based on bluetooth v4.0. The advantage is that there is no need to have additional hardware, e.g. via GPIO connected sender/receiver. 

Voltcraft SEM-3600BT / Wittech WiTenergy E100 has the following features:
* 6 schedulers which can run once or assigned to weekdays
* Countdown mode in order to switch power _on_ or _off_ after a certain period 
* Configurable overload-mode in order to auto-turn-off or alarm in case of power overload
* Configurable standby-mode in order to out switch off in case of low consumption
* Energy meter in order to measure volt, ampere and watts in realtime
* Energy meter recorder in order to measure power consumpton over long period

For official manual by Voltcraft visit [Conrad](http://www.produktinfo.conrad.com/datenblaetter/675000-699999/684997-an-01-ml-VOLTCRAFT_SEM_3600BT_SMART_E_de_en_fr_nl.pdf)


## Getting started

### 1. Check pre-conditions

Install `expect`:
```
$ sudo apt install expect
```

Check if `gatttool` is available:
```
$ gatttool
Usage:
  gatttool [OPTION...]
...

```

### 2. Discover the MAC address of the smart energy meter

```
$ sudo hcitool lescan
E Scan ...
D0:39:72:BB:AE:EC WiT Power Meter
20:CD:39:1F:EC:DE WiT Power Meter
```

The devices are called "WiT Power Meter". Since I have two of these, there are two mac addresses

### 3. Pair bluetooth

There is no need to pair device.


### 4. Aliases
For convenience reasons I recommend to use aliases. Instead of entering the bluetooth mac address each time you want to run the script, you can call the script by using meaningful names.

The script tries to read a file called `.known_sems` which must be located in your home folder. It is a text file with two columns:

1. MAC address
2. Meaningful name

My `.known_sems` looks as follows:
```
$ cat ~/.known_sems
D0:39:72:BB:AE:EC Entertainment
20:CD:39:1F:EC:DE Lamp
```

This enables you to call the script like this
```
$ ./vc-sem.exp Lamp -sync
```

instead of 
```
$ ./vc-sem.exp 20:CD:39:1F:EC:DE -sync
```

**Note**: 
You don't even have to write the whole alias. This works as well:
```
$ ./vc-sem.exp Lam -sync
```

## Getting help

In order to get an overview of the full feature set enter the following:
```
$ ./vc-sem.exp -help
Usage: <mac/alias> -<command1> <parameters...> -<command2>
                                   <mac>: bluetooth mac address of bulb
                                   <alias>: you can use alias instead of mac address
                                            after you have run setup (see setup)
                                   <command>: For command and parameters


Basic commands:

 -on                                - turn on socket
 -off                               - turn off socket
 -toggle                            - toggle socket

Scheduler commands:

 -scheduler <n> <on|off> <hh:mm|+mm> <on|off> <hh:mm|+mm> [<smtwtfs>]
                                      n         - given scheduler where n is 1 - 6
                                      on|off    - action of scheduler at start
                                      hh:mm|+mm - start time or minutes from now
                                      on|off    - action of scheduler at end
                                      hh:mm|+mm - end time or minutes after start time, at least 1 minute
                                      smtwtfs   - (optional) weekdays, use capital letters to set, e.g. sMTWTFs
                                                  use the word "sameday" for same weekdays like today
                                                  if weekdays are missing then scheduler runs only once

 -scheduler <n> <set|unset|reset>   - activate, deactivate or reset given scheduler

 -scheduler <n>                     - query status of given scheduler where n is 1 - 6

 -scheduler reset                   - resets all schedulers, i.e. 1 - 6

 -scheduler                         - query status of all schedulers, i.e. 1 - 6

Countdown commands:

 -countdown <on|off> <hh:mm|+mm>    - set countdown action and runtime
                                      on|off - action of countdown
                                      hh:mm  - given ETA
                                      +mm    - given runtime in minutes

 -countdown <reset>                 - reset countdown

 -countdown                         - query status of countdown

Power commands:

 -overload <watts> <warn|off> <alarm>
                                    - set overload with high-water-mark and actions
                                      watts - high-water-mark, max. 3699, e.g. 3699 for 3699W
                                      warn  - led blinks when overloaded
                                      off   - turn socket off if overloaded
                                      alarm - (optional) alarm if overloaded

 -overload off                      - turn overload off

 -overload                          - query overload settings

 -standby <watts> <warn|off>        - set standby mode if consumption is below water-mark
                                      watts - low-water-mark, max. 30.00, e.g. 5.0 for 5.0W
                                      warn - led blinks when standby detected
                                      off  - auto turn off socket when standby detected

 -standby off                       - turn standby mode off

 -standby                           - query standby settings

Measurement commands:

 -measure                           - take and print measurement incl. voltage, ampere, watts, power factor and frequency
 -watch [<s>]                       - watch measurements, optional duration in seconds (use 0 for single), otherwise forever

Other commands:

 -print                             - print gathered information
 -sync                              - synchronize time
 -sleep <n>                         - pause processing for n seconds
 -verbose                           - print information about processing
 -debug                             - print actions in gatttool
 -help [<command>]                  - print general help or help for specific command
```

You get specific help for a command if you ask for it explicitly:
```
$ ./vc-sem.exp -help toggle
Usage: <mac/alias> -<command1> <parameters...> >-<command2>
                                   <mac>: bluetooth mac address of bulb
                                   <alias>: you can use alias instead of mac address
                                            after you have run setup (see setup)
                                   <command>: For command and parameters

 -toggle                            - toggle socket
```


## Basic commands

### Turn switch on and off

You can turn on the socket as follows:
```
$ ./vc-sem.exp Lamp -on
```

Note that there isn't any feedback. 

You can turn the device off by entering this:
```
$ ./vc-sem.exp Lamp -off
```

If you want to toggle the switch you use the _toggle_ command:
```
$ ./vc-sem.exp Lamp -toggle
```

### Queueing commands / sleep command

Since it takes some time to establish the bluetooth connection each time you start the script, I have introduced command queing. Each commands starts with a dash. In this example I queue 7 commands. The _sleep_ commands pauses processing before the next command starts. 

```
$ ./vc-sem.exp Lamp -on -sleep 1 -toggle -sleep 1 -toggle -sleep 1 -off
```

### Print gathered information

In most cases the script won't print anything. If you want to print some output, e.g. information that has been gathered by previous commands, you must tell the script to do so explicitly. 

```
$ ./vc-sem.exp Lam -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      no
        Connected:        no
        Alias:            Lamp
```

In this example there are just basic information available since we haven't really requested anything. 

### See power state of the socket

If you want to see if the socket is turned on and what the current power consumption is, you must ask for a measurement first and print it afterwards:

```
$ ./vc-sem.exp Lam -measure -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Measurement:
          Power:          on
          Countdown:      off
          Voltage:        236.3 V
          Ampere:         0.0 A
          Watts:          0.0 W
          Frequency:      49.98 Hz
          Power factor:   1.0
```

## Schedulers

The socket has 6 schedulers that can be programmed once or per weekday. 

First let's take a look at the current state of schedulers:
```
$ ./vc-sem.exp Lam -scheduler 1 -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Scheduler 1:
          Active:         on
          Begin:          20:15 turn on
          End:            21:45 turn off
          Weekdays:       S_____S
```

_Note, that I have queued two commands again, i.e. "-schedulers 1" and "-print". The first command gathers the information but doesn't output anything. The second command prints the gathered information._

The scheduler 1 is already set. It runs on Sunday and Saturday.

Let's check all schedulers in order to find a slot that isn't used yet:
```
$ ./vc-sem.exp Lam -scheduler -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Scheduler 1:
          Active:         on
          Begin:          20:15 turn on
          End:            21:45 turn off
          Weekdays:       S_____S

        Scheduler 2:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______

        Scheduler 3:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______

        Scheduler 4:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______

        Scheduler 5:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______

        Scheduler 6:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______
```

Ok, let's set the second scheduler for weekdays from Monday to Friday. The scheduler should turn my lamp on at 7:00 a.m. After 20 minutes it should turn off. I also want to print the result- The command looks like this:

```
$ ./vc-sem.exp Lam -scheduler 2 on 07:00 off +20 _MTWTF_ -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Scheduler 2:
          Active:         on
          Begin:          07:00 turn on
          End:            07:20 turn off
          Weekdays:       _MTWTF_
```

Instead of setting the end time of 07:20 a.m. I have used an offset of +20 minutes which is based on the start time. Of course you can also set the time explicitly. 

You can also use an offset for the start time. For example you can start a scheduler that runs in 5 Minutes from now for one hour _once today_ like this:

```
$ ./vc-sem.exp Lam -scheduler 3 on +5 off +60
```

Let's take a look at the settings of scheduler 3:

```
$ ./vc-sem.exp Lam -scheduler 3 -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Scheduler 3:
          Active:         on
          Begin:          08:38 turn on
          End:            09:38 turn off
          Weekdays:       _______
```

In order to reset a specific scheduler or all schedulers run:

``` 
$ ./vc-sem.exp Lam -scheduler 3 reset
$ ./vc-sem.exp Lam -scheduler reset
```

### Countdown

The socket has a timer. You can set it with an ease like this:
```
$ ./vc-sem.exp Lam -countdown off +5
```

This activates a countdown timer which runs for 5 minutes and then turns the socket off. You can also specify a _time of arrival (ETA)_ like this:

```
$ ./vc-sem.exp Lam -countdown off 09:45
```

Let's ask for information about the countdown that we have started before:
```
$ ./vc-sem.exp Lam -countdown -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Measurement:
          Power:          on
          Countdown:      on
          Voltage:        235.3 V
          Ampere:         0.034 A
          Watts:          4.249 W
          Frequency:      50.01 Hz
          Power factor:   0.516

        Countdown:        on
          Action:         on
          Remaining:      22:40:53
          ETA:            09:44
```

Enter the following in order to stop the running countdown:
```
$ ./vc-sem.exp Lam -countdown reset
```

## Power commands

The smart energy meter monitors power consumption continuously. It can perform actions when power consumption falls below or exceeds a certain limit.

### Overload mode

You can define an action in case that the smart energy meter exceeds a limit in terms of power consumption. In this example the socket turn off immediately in case that power consumption gets higher that 1000 Watts:

```
$ ./vc-sem.exp Lam -overload 1000 off -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Overload:
          Threshold:      1000 W
          Action:         turn off
          Alarm:          no
```

If you also want to hear an acustic alarm then add the _alarm_ parameter:
```
$ ./vc-sem.exp Lam -overload 1000 off alarm -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Overload:
          Threshold:      1000 W
          Action:         turn off
          Alarm:          yes
```

If you just want to get a visual warning - led blinks red - then it goes like this:
```
$ ./vc-sem.exp Lam -overload 1000 warn -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Overload:
          Threshold:      1000 W
          Action:         blink
          Alarm:          no
```

By entering this you turn off overload:
```
$ ./vc-sem.exp Lam -overload off
```

### Standby mode

You can also set an action if the power consumption falls unter a given limit. If you just want to be warned (led blinks), e.g. in case that consumption falls under 3 Watts, enter the following:

```
$ ./vc-sem.exp Lam -standby 3 warn
```

You can also specify that the socket turn off immediately like this:
```
$ ./vc-sem.exp Lam -standby 2 off
```

In order to turn off standby mode enter the following:
```
$ ./vc-sem.exp Lam -standby off
```

## Measurements

### Snapshot

If you want to take and print a single measurement you just have to run the script like this:

```
$ ./vc-sem.exp Lamp -measure -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lamp

        Measurement:
          Power:          off
          Countdown:      on
          Voltage:        237.4 V
          Ampere:         0.0 A
          Watts:          0.0 W
          Frequency:      50.01 Hz
          Power factor:   1.0
```

### Realtime monitoring

The _watch_ command captures the power data for a given period or infinitely. This monitors the power consumption for 5 seconds:
```
$ ./vc-sem.exp Lamp -watch 5
2019-05-25 11:41:40;1;1;237.7;0.035;4.322;49.99;0.514
2019-05-25 11:41:41;1;1;237.6;0.035;4.304;49.99;0.512
2019-05-25 11:41:42;1;1;237.7;0.035;4.306;49.99;0.513
2019-05-25 11:41:43;1;1;237.7;0.035;4.317;49.99;0.514
```

Note that the _watch_ command directly prints a csv record. If you have left out the period, here "5" for 5 seconds, it runs forever. You can stop it by pressing CTRL-C

Each record has the following information:
1. Column: Timestamp
1. Column: Power state, 1 = on, 0 = off
1. Column: Countdown state, 1 = running, 0 = off
1. Column: Volt (V)
1. Column: Ampere (A)
1. Column: Watts (W)
1. Column: Frequency (Hz)
1. Column: Power factor (whatever this means)


## Request recorded data

### Power-on uptime

The smart energy meter records the uptime - the time when the socket is turned on.

You can request this time by using the _uptime_ command:

```
$ vc-sem Ent -uptime -print

        Mac:              D0:39:72:BB:AE:EC
        Initialized:      yes
        Connected:        yes
        Alias:            Entertainment

        Power-on uptime:  2 days, 20:38
```


### Power-on consumption

The smart energy meter records the overall power consumption. This can be queried like this:

```
$ vc-sem Ent -consumption -print

        Mac:              D0:39:72:BB:AE:EC
        Initialized:      yes
        Connected:        yes
        Alias:            Entertainment

        Power-on consumption:  0.109 kWh
```

### Reset recorded data

If you want to reset all recorded data enter the following:
```
$ vc-sem Ent -reset
```

## More commands

### Synchronize time

Actually time is synchronized each time you run a command. If you just want to synchronize the time from your PC with the smart meter call:

```
$ ./vc-sem.exp Lamp -sync
```

### Device information

In order to get general device information you can request it by using the _device_ command:
```
$ vc-sem La -device -print

        Mac:              20:CD:39:1F:EC:DE
        Device name:      WiT Power Meter
        Device vendor:    Wittech Company Ltd.
        Device serial:    SN: 000000
        Device firmware:  F/W: V01.32
        Device hardware:  H/W: V00.00
        Device software:  S/W: V00.11
        Device secret:    0x030c
```

### Verbose

If you want to get some information what's going on while the script is running, add the verbose command. 

```
$ ./vc-sem.exp Lamp -verbose -countdown off +10 -measure -off
INFO:   Try to connect to 20:CD:39:1F:EC:DE
INFO:   Connected to 20:CD:39:1F:EC:DE
INFO:   Sync SEM
INFO:   >>>     char-write-req 18 03e30705190b23340c03
INFO:   OK
INFO:   SEM synced successfully
INFO:   Set countdown
INFO:   >>>     char-write-req 18 06000a
INFO:   OK
INFO:   Countdown successfully set
INFO:   Subscribe for notification handle 0x13
INFO:   >>>     char-write-req 13 0100
INFO:   OK
INFO:   Successfully subscripted to notification handle 0x13
INFO:   Wait for notification (5)
INFO:   <<<     Notification handle = 0x0012 value: 02 03 23 83 01 00 00 01 00 00 01 10 00 02 49 97
INFO: Handle 18, Bytes 2 3 35 131 1 0 0 1 0 0 1 16 0 2 73 151
        Measurement:
          Power:          off
          Countdown:      on
          Voltage:        238.3 V
          Ampere:         0.0 A
          Watts:          0.0 W
          Frequency:      49.97 Hz
          Power factor:   1.0


INFO:   Measurement successfully captured
INFO:   Switch SEM off
INFO:   >>>     char-write-req 18 0400
INFO:   OK
INFO:   SEM successfully switched
INFO:   Disconnect from 20:CD:39:1F:EC:DE
```

### Debug

The script uses _gatttool_ from the _bluez_ bluetooth stack. You can also see the commands sent to gatttool by using the _debug_ command:

```
$ ./vc-sem.exp Lamp -debug -countdown off +10 -measure -off
[                 ][LE]> connect 20:CD:39:1F:EC:DE
Attempting to connect to 20:CD:39:1F:EC:DE
Connection successful
[20:CD:39:1F:EC:DE][LE]> char-write-req 18 03e30705190b26310c03
Characteristic value was written successfully
[20:CD:39:1F:EC:DE][LE]> char-write-req 18 06000a
Characteristic value was written successfully
[20:CD:39:1F:EC:DE][LE]> char-write-req 13 0100
Characteristic value was written successfully
Notification handle = 0x0012 value: 02 03 23 82 01 00 00 01 00 00 01 10 00 02 49 98
[20:CD:39:1F:EC:DE][LE]> char-write-req 18 0400
Characteristic value was written successfully
```

### Dump

The _dump_ command tries to gather all information from the device.
```
$ ./vc-sem.exp La -dump -print
```