# voltcraft-sem-3600bt
Full-features shell script in order to manage Voltcrafts's bluetooth smart energy meter, switch and scheduler with Linux and Raspberry Pi

The Voltcraft SEM-3600BT is a remote 230V switch and smart energy meter. It was sold by Conrad Elektronic in Germany a few years ago. For details take a look at [Amazon](https://www.amazon.de/Voltcraft-SEM-3600BT-Energiekosten-Messger%C3%A4t-Bluetooth%C2%AE-Schnittstelle-Darstellung/dp/B00IWS7V8G)

In other countries it is probably known as Wittech WiTenergy E100 

In comparison to many remote switches which use the 433MHz band the SEM-3600BT is based on bluetooth v4.0. The advantage is that there is no need to have additional hardware, e.g. connected via GPIO. 

Voltcraft has the following features:
* 6 schedulers which can be individually assigned to weekdays or run once
* Countdown mode in order to switch power on or off after a certain period 
* Configurable power-off and alarm mode in case of overload
* Configurable power-off mode in case of low consumption, e.g. devices in standby
* Energy meter in order to measure volt, ampere and watts 

For official manual by Voltcraft visit [Conrad](http://www.produktinfo.conrad.com/datenblaetter/675000-699999/684997-an-01-ml-VOLTCRAFT_SEM_3600BT_SMART_E_de_en_fr_nl.pdf)


## Getting started

0. Check pre-conditions

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

1. Discover the MAC address of the smart energy meter

```
$ sudo hcitool lescan
E Scan ...
D0:39:72:BB:AE:EC WiT Power Meter
20:CD:39:1F:EC:DE WiT Power Meter
```

The devices are called "WiT Power Meter". Since I have two of these, there are two mac addresses

2. Pair bluetooth

There is no need to pair devices. 

## Aliases
For convenience reasons I recommend to use aliases. Instead of entering the mac address each time you want to run the script, you can call the script by using meaningful names. 

The script tries to read a file called `.known_sems` which must be located in your home folder. It is a text file with two columns:
1. MAC address
2. Meaningful name

My `.known_sems` looks as follows:
```
$ cat ~/.known_sems
D0:39:72:BB:AE:EC Entertainment
20:CD:39:1F:EC:DE Lampe
```

This enables you to call the script like this
```
$ ./vc-sem.exp Lampe -sync
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
Usage: <mac/alias> -<command1> <parameters...> >-<command2>
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
                                      on|off    - action of countdown
                                      hh:mm|+mm - runtime of countdown

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
$ ./vc-sem.exp Lampe -on
```

There is no feedback. 

You can turn the lamp off by entering this:
```
$ ./vc-sem.exp Lampe -off
```

If you want to toggle you use the _toggle_ command:
```
$ ./vc-sem.exp Lampe -toggle
```

### Print some information

In order to print information especially information that has been requested make use of the _print_ command:

```
$ ./vc-sem.exp Lam -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      no
        Connected:        no
        Alias:            Lampe
```

In this example there are just the basics since we haven't really requested anything. 

### See power state of the socket

If you want to see if the socket is turned on and what the power consumption is, you must ask for a measurement and print it afterwards:

```
$ ./vc-sem.exp Lam -measure -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

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
        Alias:            Lampe

        Scheduler 1:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______


$ ./vc-sem.exp Lam -scheduler -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

        Scheduler 1:
          Active:         off
          Begin:          00:00 turn off
          End:            00:00 turn off
          Weekdays:       _______

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

Note, that I have actually queued _two_ commands, i.e. "-schedulers" and "-print". The first command gathers the information from the socket but doesn't output anything. That's why you need to ask explicitly for printing the gathered information. 

Let's set the first scheduler for weekdays from Monday to Friday. The scheduler should turn my lamp on at 7:00 a.m. After 20 minutes it should turn off. I also want to print the result- The command looks like this:

```
$ ./vc-sem.exp Lam -scheduler 1 on 07:00 off +20 _MTWTF_ -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

        Scheduler 1:
          Active:         on
          Begin:          07:00 turn on
          End:            07:20 turn off
          Weekdays:       _MTWTF_
```

Instead of setting the end time of 07:20 a.m. I have used an offset of +20 minutes which is based on the start time. Of course you can also set the time explicitly. 

You can also use an offset for the start time. For example you can start a scheduler that runs in 5 Minutes from now for one hour _once today_ like this:

```
$ ./vc-sem.exp Lam -scheduler 1 on +5 off +60
```

Let's take a look at the settings of scheduler 1:

```
$ ./vc-sem.exp Lam -scheduler 1 -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

        Scheduler 1:
          Active:         on
          Begin:          08:38 turn on
          End:            09:38 turn off
          Weekdays:       _______
```

In order to reset a specific scheduler of all schedulers run:

``` 
$ ./vc-sem.exp Lam -scheduler 1 reset
$ ./vc-sem.exp Lam -scheduler reset
```

### Countdown

The socket has a countdown timer. You can set it with an ease like this:
```
$ ./vc-sem.exp Lam -countdown off +5
```

This activates a countdown timer which runs for 5 minutes and then turns the socket off. You can also specify a runtime like this:

```
$ ./vc-sem.exp Lam -countdown off 09:45
```

Ask for information about an already running countdown like this: 
```
$ ./vc-sem.exp Lam -countdown -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

        Measurement:
          Power:          on
          Countdown:      on
          Voltage:        235.3 V
          Ampere:         0.034 A
          Watts:          4.249 W
          Frequency:      50.01 Hz
          Power factor:   0.516

        Countdown:        on
          Action:         off
          Remaining:      09:44:28
          ETA:            18:28
```

In order to stop the countdown you enter this:
```
$ ./vc-sem.exp Lam -countdown reset
```

## Power commands

The smart energy meter is of course able to monitor the power consumption. It can perform actions when power consumption falls below or exceeds a certain limit. 

### Overload 

You can define an action in case that the smart energy meter exceeds a limit in terms of power consumption. In this example the socket turn immediately of if power consumption gets higher that 1000 Watts:

```
$ ./vc-sem.exp Lam -overload 1000 off -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

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
        Alias:            Lampe

        Overload:
          Threshold:      1000 W
          Action:         turn off
          Alarm:          yes
```

if you just want to see a warning - the led blinks red - then it goes like this:
```
$ ./vc-sem.exp Lam -overload 1000 warn -print

        Mac:              20:CD:39:1F:EC:DE
        Initialized:      yes
        Connected:        yes
        Alias:            Lampe

        Overload:
          Threshold:      1000 W
          Action:         blink
          Alarm:          no
```

Turn off overload by entering this:
```
$ ./vc-sem.exp Lam -overload off
```

### Standby mode

You can also set an action if the power consumption falls unter a limit. If you just want to be warned, e.g. in case that consumption falls under 3 Watts, enter the following:

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


## Other commands

