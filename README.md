<h1 align="center">
  <br>
<img src="./assets/icon.png" alt="icon" width="200">
  <br>
  NVFanControl
  <br>
</h1>

<h4 align="center">An Nvidia Fan Control service for Wayland, designed for NixOS.</h4>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#requirements">Requirements</a> •
  <a href="#installation">Installation</a> •
  <a href="#configuration">Configuration</a> •
  <a href="#removal">Removal</a> •
  <a href="#credits-and-thanks">Credits and Thanks</a> •
  <a href="#license">License</a>
</p>

> [!WARNING]  
> This repository is shared without warranty, as detailed in the license. Use at your own risk.

> [!CAUTION]  
> If uninstalling, be sure to re-enable automatic fan control with
> ```
> sudo nvidia-settings -a gpufancontrolstate=0
> ```


## Overview

NVFanControl is a simple service that checks the current temperature of the Nvidia GPU and adjusts the fan speed according to a pre-set fan curve. I wrote this because existing solutions required manually running sudo scripts, and often handled passwords dubiously. 

This is written for NixOS, but can be easily adopted to most Unix-based operating systems.


## Requirements

* Nvidia-settings must be installed.
* You must have a working understanding of NixOS configuration files. 


## Installation

Copy NVFanControl.nix to your /etc/nixos directory, and add it as an import to your primary configuration.nix with:

```
 imports =
    [ 
      ./NVFanControl.nix
    ];
```

After rebuilding, the service should be running automatically. You can check the status with:
```
systemctl status NVFanControl
```
The system should return the current temperature and target fan speed. It will also confirm that the target fan speed was assigned to the GPU fan(s).

```
Starting NVIDIA Fan Control Bash Script...
Current Temp: 41 | Target Fan Spped: 30
Attribute 'GPUFanControlState' ([gpu:0]) assigned value 1.
Attribute 'GPUTargetFanSpeed' ([fan:0]) assigned value 30.
Attribute 'GPUTargetFanSpeed' ([fan:1]) assigned value 30.
NVFanControl.service: Deactivated successfully.
```

Some systems may present errors about a missing libEGL. These errors seem to originate from nvidia-settings and seem to be ignorable.


## Configuration

By default, the script will check the current temperature of the GPU on Display 0 every second, and adjust the fan speed accordingly. The curve smoothly raises the the fan speed from 30% at <= 50 degrees C, to 100% at 80 degrees C.

No direct configuration is supported, but the script is small and easily modified for your use-case. Points of interest might include:
* /run/current-system/sw/bin is only need on NixOS systems. Traditional systems can omit this.
* The -c 0 option of nvidia-settings refers to the display to be controlled (e.g. DISPLAY=:0).
* The script will disable automatic fan control on every run with the nvidia-settings option GPUFanControlState=1. 
* The script uses a steep sine wave curve to smoothly ramp from 50. If you want to adjust this curve, use a graphing calculator to check your formula. Pay attention to the temperature floor and ceilling, as your curve may oscillate a different direction than intended if it is not properly bound. Test changes thoroughly.


## Removal

> [!CAUTION]  
> Pausing or uninstalling the script will leave your GPU fans spinning at the last set speed. Automatic fan control will be disabled. To re-enable fan control after pausing or uninstall the service, use your GUI or:
> ```
> sudo nvidia-settings -a gpufancontrolstate=0
> ```
 
Removing the import from your configuration.nix and rebuilding should remove the service from your system.

To temporarily disable the script, you can stop the timer with systemctl stop NVFanControl.timer. This will prevent NVFanControl from triggering.


## Credits and Thanks

- Readme.md based on <a href="https://www.readme-templates.com">Readme-Template</a> by <a href="https://github.com/amitmerchant1990"> Amit Merchant</a>

## License

MIT
