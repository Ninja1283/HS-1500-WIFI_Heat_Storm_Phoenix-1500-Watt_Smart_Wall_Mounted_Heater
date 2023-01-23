# HS-1500-WIFI Heat Storm Phoenix 1500 Watt Smart Wall Mounted Heater

Instructions, tips, and information for obtaining Tuya DP IDs, flashing custom ESPHome firmware, and integrating the Heat Storm Phoenix 1500 Watt Smart Wall Mounted Heater with Home Assistant.

DPs and ESPHome yaml below. See the Wiki for more info and full wrietup.

## Product Information

The product in question is [this](https://heatstorm.com/collections/wall-heaters/products/phoenix-wifi-smart-heater) Wifi-enabled 1500W infrared heater. Product manual can be found [here](https://cdn.shopify.com/s/files/1/1873/7325/files/HS-1500PHX-WIFI_Manual-compressed.pdf).

## DP IDs

|Description|DP Code|Data Type|DP ID|Values|
|:-|:-|:-|:-|:-|
|Switch|switch|Boolean|1|0=off,1=on|
|Set Temperature (°C)|temp\_set|Integer|2|Int range (4-37)|
|Current Temperature (°C)|temp\_current|Integer|3|Int range (-9-37)|
|Child Lock|lock|Boolean|7|0=on, 1=off|
|Temperature Scale|temp\_unit\_convert|Enum|19|0=C, 1=F|
|Set Temperature (°F)|temp\_set\_f|Integer|20|Int range (40-99)|
|Current Temperature (°F)|temp\_current\_f|Integer|21|Int range (16-99)|
|Mode|mode|Enum|4|0=auto, 1=low, 2=high|
|Brightness|level|Enum|5|0=off, 1=10%, 2=50%, 3=100%|
|Fault Alarm|fault|Boolean|13|0=clear, 1=fault|

## ESPHome yaml

[heater.yaml](docs/heater.yaml)


## Acknowledgements

@ssieb for their insight on using the climate component in ESPHome

[EnsignR](https://community.home-assistant.io/u/ensignr) in the HA forums for their helpful tips on obtaining Tuya DP IDs and Local Keys.
