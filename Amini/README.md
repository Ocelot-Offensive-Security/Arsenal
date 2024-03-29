# Amini Project

The project is divided in three different sections:
* ArduinoBLE
* examples
* hardware



## ArduinoBLE
The ArduinoBLE contains the modified Bluetooth BLE library to handle the Apple AirTags advertising packages and special features regarding scanning, detection and spoofing. To use this library, just copy and paste it into the Arduino library section.

Some of the most important changes in the Arduino BLE library could be found into the advertising packages handler.

To obtain the advertising packages data at BLEDevice.cpp:
```
int BLEDevice::getAdvertisement(uint8_t value[], int length)
{
  if (_eirDataLength) {
    memcpy(value, _eirData, length);
  }

  return _eirDataLength;
}
```
To be able to advertise an AirTag package, we modified the flags assignation at local/BLELocalDevice.cpp:
```
BLELocalDevice::BLELocalDevice()
{
  //_advertisingData.setFlags(BLEFlagsGeneralDiscoverable | BLEFlagsBREDRNotSupported);
}
```

## Examples
In contains different examples to be used with supported Arduino-environment boards.
- leds: detecting AirTag presence by flashing LEDs depending on signal strength
- scan: continuous scanning to detect enabled and disabled AirTags 
- sound: detecting enabled AirTags and connected to them to generated a register, if after 10 minutes the AirTags are nearby, Amini can force to play a sound
- spoof: simulates an AirTag behavior for packaging advertising 


## Hardware
It contains the open-hardware board presented in Black Hat USA Arsenal on Aug 11th, 2022. https://www.blackhat.com/us-22/arsenal/schedule/index.html#amini-project-27224

### LEDs and Buttons pins
LEDs:
- Pin 18: D3
- Pin 19: D6
- Pin 21: D5
- Pin 22: D4

Buttons:
- Pin 7: SW3
- Pin 8: SW4
