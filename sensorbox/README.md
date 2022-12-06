# Sensorbox V1

The goal of this project was to have several sensors which can be integrated into HomeKit through [homebridge](https://homebridge.io/).

For this I have running a pine64 Board (you can use a raspberry pi) with Linux where Homebride is installed and [mosquitto](https://mosquitto.org/download/) as MQTT broker for the network communication.

The costs for the required electronic parts of one box are under 10 €/$.

## Features

- Motionsensor
- Lux-Sensor (Illumination)
- Temperature/Huminity Sensor
- Wifi
- MQTT Network Communication

## Images / Prototypes

| Stage |  |
| ------ | ----------- |
| Prototype 1   | ![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/prototyp-1.jpg) |
| Prototype 2 | ![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/prototyp-2.jpg) |
| Prototype 3 / Final Prototype | ![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/prototyp-3.jpg) |
| Sensorbox v2 Final | ![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/sensorbox_v1.jpg) |

----
## Parts

### Mandatory

- NodeMCU Board with ESP8266 ([Amazon-Link](https://www.amazon.de/gp/product/B074Q2WM1Y))
- LUX Sensor BH1750 ([Amazon-Link](https://www.amazon.de/gp/product/B07VF15XJJ))
- Motion Sensor AM312 ([Amazon-Link](https://www.amazon.de/gp/product/B08931TTTN))
- Temperature/Humidity Sensor AM3202/DHT22 ([Amazon-Link](https://www.amazon.de/MakerHawk-Digitales-Temperatur-Feuchtigkeitsmesssensormodul-Arduino/))
- Jumper cables / cables ([Amazon-Link](https://www.amazon.de/AZDelivery-Jumper-Arduino-Raspberry-Breadboard/dp/B074P726ZR/))
- Power Supply 5V Micro USB

### Optional
- Holegrid Board to solder on ([Amazon-Link](https://www.amazon.de/gp/product/B0899K85BS/))
- Cases
    - [Plastic Housing Electronic Enclosure Case 81 * 41 * 20 mm](https://www.amazon.de/gp/product/B08F3NJ3Y2/) (A lot of work needed to get everything/no clean solution possible)
    - [LaDicha 75 x 54 x 27mm DIY](https://www.amazon.de/gp/product/B07CKRDR7W/)
    - 3D Printed DIY [Download STL File for Download](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/sensorbox_v1.stl)

---
## Scheme

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/sensorbox-scheme.jpg)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/sensorbox-scheme.jpg)

> This scheme is important for the software settings  which are described on the next section

[Download Fritzing File](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/sensorbox.fzz)

---
## Software


As firmware I flashed [Tasmota](https://tasmota.github.io/docs/) which provides the functionality we need without the need of writing our own firmware.

Here you can download the [tasmota-sensors.bin firmware](http://ota.tasmota.com/tasmota/release/tasmota-sensors.bin) which is needed to get all sensors of this project working. If you flash another version of tasmota its most likely that the BH1750 and maybe the AM312 won't work.

### Flash the firmware

You can flash the firmware via usb to the nodeMCU. I used the [nodeMCU.pyFlasher](https://github.com/marcelstoer/nodemcu-pyflasher) to flash the nodeMCU with the downlaoded firmware.

### Initial Setup

For the initial setup of tasmota [they provide a detailed documentation](https://tasmota.github.io/docs/Getting-Started/#initial-configuration). Please follow their instructions to setup your wifi and mqtt broker.

### Setup Tasmota Sensorbox

1. You need to click on configuration

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-start.png)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-start.png)

2. On the configurationscreen click configure module to setup the sensors

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration.png)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration.png)

2. On the modulconfiguration we have to setup the sensors which

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration-module.png)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration-module.png)

> this is how the configuration needs to look like when you setted up the sensors on the GPIO Pins like shown in the scheme section

3. Setup your MQTT broker for the mqtt network communication

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration-mqtt.png)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-configuration-mqtt.png)

| Entry | Description |
| ------ | ----------- |
| Host   | The IP of your mqtt broker |
| Port | The Port of your mqtt broker |
| Client    | This is a generic client > you can leave that as it is |
| User    | the user which is allowed to send date to the mqtt broker |
| Password    | the mqtt users password |
| Topic    | The Topic of the mqttClient -> in this case its sensorbox1 (Other clients can subscribe to the topic to get its values) |
| Full Topic    | The full topic which is send to the mqtt broker e.g. 'sensorbox1/sensors/' |

4. Additional enhancements

Navigate back to the main menu and open the console.

[![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-console.png)](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/tasmota-console.png)

We need to setup some rules to get the sensors working and communicating like we want it to implement them into homebrew.

>We need to set a rule for the PIR sensor so it publishes a specific event via mqtt

```
SwitchMode1 14
SwitchTopic 0
Rule1 on Switch1#state=1 do Backlog Publish %topic%/stat/PIR1 ON; RuleTimer1 120 endon on Rules#Timer=1 do Publish %topic%/stat/PIR1 OFF endon
Rule1 1
```

>We need to set a longer time to measure the light intensity and get more granular lux values

```
Rule2  ON System#Boot DO Bh1750MTime1 254 ENDON
Rule2 1
```

## Setup the sensorbox in homebridge

To setup our sensorbox on homebridge to integrate it into our homekit environment and use it for automations and so on, we need to install the [Homebridge MQTT-Thing Plugin](https://github.com/arachnetech/homebridge-mqttthing#readme) in homebridge ui.
Once the plugin is installed this is how the configurations accesories section has to look like:
> fill in your MWTT data in the areas marked with <>

``` json
{
    "bridge": {
    },
    "accessories": [
        {
            "accessory": "mqttthing",
            "type": "humiditySensor",
            "name": "Sensorbox 1 Humanity",
            "url": "mqtt://<MQTT-BROKER-IP>:<MQTT-BROKER-PORT>",
            "username": "<MQTT-BROKER-USER>",
            "password": "<MQTT-BROKER-PASSWORD>",
            "caption": "Sensorbox 1 Humanity",
            "topics": {
                "getCurrentRelativeHumidity": {
                    "topic": "sensorbox1/tele/SENSOR",
                    "apply": "return JSON.parse(message).AM2301.Humidity;"
                },
                "getStatusActive": "sensorbox1/stat/POWER",
                "getStatusFault": "sensorbox1/tele/STATE"
            },
            "history": true
        },
        {
          "accessory": "mqttthing",
          "type": "temperatureSensor",
          "name": "Sensorbox 1 Temperature",
          "url": "mqtt://<MQTT-BROKER-IP>:<MQTT-BROKER-PORT>",
          "username": "<MQTT-BROKER-USER>",
          "password": "<MQTT-BROKER-PASSWORD>",
          "caption": "Sensorbox 1 Temperature",
          "topics": {
            "getCurrentTemperature": {
              "topic": "sensorbox1/tele/SENSOR",
              "apply": "return JSON.parse(message).AM2301.Temperature;"
            }
          }
        },
        {
          "accessory": "mqttthing",
          "type": "lightSensor",
          "name": "Sensorbox 1 Lightsensor",
          "url": "mqtt://<MQTT-BROKER-IP>:<MQTT-BROKER-PORT>",
          "username": "<MQTT-BROKER-USER>",
          "password": "<MQTT-BROKER-PASSWORD>",
          "caption": "Sensorbox 1 Lightsensor",
          "topics": {
            "getCurrentAmbientLightLevel": {
              "topic": "sensorbox1/tele/SENSOR",
              "apply": "return JSON.parse(message).BH1750.Illuminance;"
            }
          }
        },
        {
          "accessory": "mqttthing",
          "type": "motionSensor",
          "name": "Sensorbox 1 Motionsensor",
          "url": "mqtt://<MQTT-BROKER-IP>:<MQTT-BROKER-PORT>",
          "username": "<MQTT-BROKER-USER>",
          "password": "<MQTT-BROKER-PASSWORD>",
          "caption": "Sensorbox 1 Motionsensor",
          "topics": {
            "getMotionDetected": "sensorbox1/stat/PIR1"
          },
          "onValue": "ON",
          "offValue": "OFF",
          "turnOffAfterms": "120000",
          "history": true
        }
    ],
    "platforms": []
}
```

> we need to setup the sensors seperately for homekit.

Once added you need to restart homebridge and if the sensorbox is running it should appear in homebridge and therefore in HomeKit aswell.

This is how the end result should look like in HomeKit:

![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/homekit-sensors.jpg)

![](https://github.com/artifactdev/iot-mqtt-homekit-projects/raw/main/sensorbox/assets/homekit-sensors-2.jpg)
