# nanopim4-satahat-fan
A fan control script written in bash for the [**2-pin PH2.0 12v fan connector of the NanoPi M4 SATA hat**](http://wiki.friendlyarm.com/wiki/index.php/NanoPi_M4_SATA_HAT). By default, the script uses a bounded [logistic model](https://en.wikipedia.org/wiki/Logistic_function) with a moving mid-point (based on the average temperature over time) to set the fan speed. 

Many of the variables used in this fan controller can be modified directly from the CLI, such as setting custom temperature thresholds (`-t`, `-T`) or disabling temperature monitoring altogether (`-f`). For a more detailed description, see [**Usage**](#usage).

There's arguably more code here than necessary to run a fan controller. This was a hobby of mine (I wanted to revisit the first version which used a fixed table to set the speed) and an opportunity to learn more about bash and the sysfs interface.  

This is free. There is NO WARRANTY. Use at your own risk. 

If you have any issues or suggestions, open an issue or [send me an e-mail](mailto:me@cgomesu.com). 


# Requisites
- Linux distro;
- Access to the [pwm sysfs interface](https://www.kernel.org/doc/Documentation/pwm.txt) (run with `sudo` permission or as `root`);
- [GNU bash](https://www.gnu.org/software/bash/) (recommend v5.x);
- [GNU basic calculator](https://www.gnu.org/software/bc/);
- Standard GNU/Linux commands.

Besides bash, you don't need to check for any of these requisites manually. The script will automatically check for everything it needs to run and will let you know if there's any errors or missing access to important commands.  

The controller was developed with **Armbian OS** but you should be able to run it on **any other Linux distro** for the NanoPi M4. For reference, this script was originally developed with the following hardware:
-  NanoPi-M4 v2
-  M4 SATA hat
-  12V (.08A-.2A) generic fan (**do not** try to use more than **200mA**)

And software:
-  Kernel: Linux 4.4.213-rk3399
-  OS: Armbian Buster (21.08.8) stable
-  GNU bash v5.0.3
-  bc v1.07.1

<p align="center">
  <img width="350" height="350" src="img/fan-pins-500-500.jpg"><br>
  <img width="350" height="350" src="img/fan-soc-500-500.jpg">
</p>


# Installation
```
apt update
apt install git
cd /opt

# From now on, if you're not running as root, append 'sudo' if you run into permission issues
git clone https://github.com/cgomesu/nanopim4-satahat-fan.git
cd nanopim4-satahat-fan

# Allow the script to be executed
chmod +x pwm-fan.sh

# Test the script
./pwm-fan.sh

# Check for any error messages 
# When done, press Ctrl+C after to send a SIGINT and stop the script
```


# Usage
```
./pwm-fan.sh -h
```
```
Usage:

./pwm-fan.sh [OPTIONS]

  Options:
    -c  str  Name of the PWM CHANNEL (e.g., pwm0, pwm1). Default: pwm0
    -C  str  Name of the PWM CONTROLLER (e.g., pwmchip0, pwmchip1). Default: pwmchip1
    -d  int  Lowest DUTY CYCLE threshold (in percentage of the period). Default: 25
    -D  int  Highest DUTY CYCLE threshold (in percentage of the period). Default: 100
    -f       Fan runs at FULL SPEED all the time. If omitted (default), speed depends on temperature.
    -F  int  TIME (in seconds) to run the fan at full speed during STARTUP. Default: 60
    -h       Show this HELP message.
    -l  int  TIME (in seconds) to LOOP thermal reads. Lower means higher resolution but uses ever more resources. Default: 10
    -m  str  Name of the DEVICE to MONITOR the temperature in the thermal sysfs interface. Default: soc
    -p  int  The fan PERIOD (in nanoseconds). Default (25kHz): 25000000.
    -s  int  The MAX SIZE of the TEMPERATURE ARRAY. Interval between data points is set by -l. Default (store last 1min data): 6.
    -t  int  Lowest TEMPERATURE threshold (in Celsius). Lower temps set the fan speed to min. Default: 25
    -T  int  Highest TEMPERATURE threshold (in Celsius). Higher temps set the fan speed to max. Default: 75
    -u  int  Fan-off TEMPERATURE threshold (in Celsius). Shuts off fan under the specified temperature. Default: 0
    -U  int  Fan-on TEMPERATURE threshold (in Celsius). Turn on fan control above the specified temperature. Default: 1

  If no options are provided, the script will run with default values.
  Defaults have been tested and optimized for the following hardware:
    -  NanoPi-M4 v2
    -  M4 SATA hat
    -  Fan 12V (.08A and .2A)
  And software:
    -  Kernel: Linux 4.4.213-rk3399
    -  OS: Armbian Buster (21.08.8) stable
    -  GNU bash v5.0.3
    -  bc v1.07.1

Author: cgomesu
Repo: https://github.com/cgomesu/nanopim4-satahat-fan

This is free. There is NO WARRANTY. Use at your own risk.

```


# Examples
- Run with a custom period and min/max temperature thresholds
```
./pwm-fan.sh -p 25000000 -t 30 -T 60
```

- Run with defaults, except that the minimum duty cycle threshold is 40%
```
./pwm-fan.sh -d 40
```

- Run in full speed mode all the time
```
./pwm-fan.sh -f
```

- Set fan startup to 10 sec
```
./pwm-fan.sh -F 10
```

- When using args `-u` and `-U` (introduced by [@araynard](https://github.com/araynard) via [#7](https://github.com/cgomesu/nanopim4-satahat-fan/pull/7)), it is recommended to leave a difference of at least 5°C between them.  In most cases, `-u` can be set to a value slightly higher than the idle temperature *with* the fan, whereas `-U` can be set to a value slightly higher than the idle temperature *without* the fan.
```
./pwm-fan.sh -u 45 -U 55
```


# Run in the background
If you're running options different than the default values, first edit the `pwm-fan.service` file to include those options into the `ExecStart=` command execution. 

```
# Enable the service and start it
systemctl enable /opt/nanopim4-satahat-fan/pwm-fan.service
systemctl start pwm-fan.service

# Check the service status to make sure it's running without issues
systemctl status pwm-fan.service
```
