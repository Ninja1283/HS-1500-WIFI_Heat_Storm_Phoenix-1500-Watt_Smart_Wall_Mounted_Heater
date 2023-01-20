# HS-1500-WIFI_Heat_Storm_Phoenix-1500-Watt_Smart_Wall_Mounted_Heater
Instructions, tips, and tricks for obtaining DP IDs, flashing, and working with the Heat Storm Phoenix 1500 Watt Smart Wall Mounted Heater with Home Assistant

## Background:
I've tried to avoid Tuya devices (and cloud-based devices in general) when making IOT product selections, but by the end of the recent holiday season, I ended up with two Tuya devices: a Sous Vide stick, and a infrared heater. (For the purpose of this project, we'll focus on the heater, and address the sous vide in a later project.)

I briefly tried using the HA Tuya Integrations, but both options seemed a bit clunky and limiting, and the Sous Vide completely disconnected twice. The second time this happened, I decided it was time to reflash both devices.

I learned a bit along the way, and thought I would share my experiences, as many of existing guides and tips that I found are either confusing, spread out across multiple sources, or out of date.

## Integration Options
In order to connect your Tuya devices with Home Assistant, you have three primary options:
1. Stock firmware and [Official Tuya Integration](https://www.home-assistant.io/integrations/tuya/) (Cloud Based)
2. Stock firmware and [LocalTuya Integration](https://github.com/rospogrigio/localtuya) (Local)
3. Custom ESPHome/Tasmota firmware, and [ESPHome](https://esphome.io/)/[Tasmota](https://tasmota.github.io/docs/) Integration

### Official Home Assistant Tuya Integration
If you plan on using the Official (Cloud Based) Tuya Integration in Home Assistant, you'll need to set up a Tuya IoT Platform Account (see link above for full instructions.) To set up this Integration, you'll need to know your Tuya IoT Access ID, Tuya IoT Access Secret, and the email/password that you use to log in to the Smart Life or Tuya phone app, and add your Tuya device(s) to one of these two apps. This is by far the easiest way to get your Tuya devices working with Home Assistant, but it also means that they are still reliant on a cloud service to work.
![Official Tuya Integration Setup Screen](https://imgur.com/kivCL4f)

### LocalTuya Home Assistant Tuya Integration
If you want a local-only option, there is a third-party Tuya Integration available for Home Assistant. It does require a bit more to set up, but frees you from the cloud. There are full instructions in the link above, but in a nutshell, you'll still need to set up a Tuya IoT Platform Account like the Official Tuya Integration and add your device(s) to the phone app. Additionally, you also need to know the following information about each device:
- Local Key
- Host ( Device IP Address on your local network)
- Device ID
- Protocol Version
- DP IDs (in most cases, the Local Tuya Integration will automatically retreive these IDs for you, but you'll still need to know which DP ID corresponds to which command; ie DP #1 turns your device on/off, DP #2 sets the temperature, etc.)
![Local Tuya Integration Setup Screen](https://imgur.com/1EhckPy)

### Flashing your own firmware with Tasmota or ESPHome
If you want to unlock the full potential of your device, and customize it however you see fit, both the previously linked Tasmota and ESPHome projects will allow you to do this. (For the purpose of this project, I'll be using ESPHome, but Tasmota is another option that will work.) Basically, you will replace the stock Tuya firmware running on your device, and add it to Home Assistant using the ESPHome Integration. While setting up a Tuya IoT Platform Account isn't strictly necessary for this setup, you will still need to know your device DP IDs, and which controls they are mapped to, so setting up a Tuya account does make this this process much easier by taking out all of the guesswork.

## Hardware and Device Specific Information
The product in question is [this](https://heatstorm.com/collections/wall-heaters/products/phoenix-wifi-smart-heater) Wifi-enabled 1500W infrared heater. Product manual can be found [here](https://cdn.shopify.com/s/files/1/1873/7325/files/HS-1500PHX-WIFI_Manual-compressed.pdf).

Like many Tuya IOT devices, this unit has one Tuya MCU to handle the actual machine operation, and a second Wifi MCU to handle wireless duties, communicating back and forth to the main MCU via serial, and to the Tuya cloud servers or your local Home Asssistant instance via Wifi. Since this device uses a secondary Tuya MCU, it is important that we map out the DP IDs before disassembly or flashing. Regardless of whether you plan on going with the LocalTuya or ESPHome/Tasmota options, you'll need to get some information first.

### Obtaining DP IDs and Local Key
There are a pair of good guides on how to obtain the DP IDs [here](https://linkdhome.com/articles/local-tuya-device-control-in-homekit) and [here](https://www.zigbee2mqtt.io/advanced/support-new-devices/03_find_tuya_data_points.html). The first method is a bit eaiser, but I've found that it often misses some of the DP IDs.

Once you have your account setup and your devices added, select "Cloud" --> "Development" from the left menu:
![Development](https://i.imgur.com/hHxBilu.png)

From there, select your project, and you will be taken to the "Overview":
![Overview](https://i.imgur.com/8RxniO2.png)

Select the "Devices" tab at the far right:
![Devices](https://i.imgur.com/8RxniO2.png)

Record your "Device ID". You'll need it for the next steps. If you're using the Local Tuya Integration, you'll also need to enter it when setting up the Integration in HA.

Click on the "Debug Device" Link at the far right:
![Debug Device](https://i.imgur.com/rqRclTH.png)

Next, navigate to "Cloud" --> "API Explorer":
![API Explorer Navigation](https://i.imgur.com/JiSNpJh.png)

This should open the API Explorer in a new window/tab:
![API Explorer Window](https://i.imgur.com/kH16TVv.png)

Enter the "Device ID" you recorded in the previous step
