# HS-1500-WIFI Heat Storm Phoenix 1500 Watt Smart Wall Mounted Heater
Instructions, tips, and tricks for obtaining Tuya DP IDs, flashing, and integrating the Heat Storm Phoenix 1500 Watt Smart Wall Mounted Heater with Home Assistant

## Background
I've tried to avoid Tuya devices (and cloud-based devices in general) when making IOT product selections, but by the end of the recent holiday season, I ended up with two Tuya devices: a Sous Vide stick, and a infrared heater. (For the purpose of this project, we'll focus on the heater, and address the sous vide in a later project.)

I briefly tried using the HA Tuya Integrations, but both options seemed a bit clunky and limiting, and the Sous Vide completely disconnected twice. The second time this happened, I decided it was time to reflash both devices.

I learned a bit along the way, and thought I would share my experiences, as many of existing guides and tips that I found are either confusing, spread out across multiple sources, or out of date.

## Integration Options
In order to connect your Tuya devices with Home Assistant, you have three primary options:
1. Stock firmware and [Official Tuya Integration](https://www.home-assistant.io/integrations/tuya/) (Cloud Based)
2. Stock firmware and [LocalTuya Integration](https://github.com/rospogrigio/localtuya) (Local)
3. Custom ESPHome/Tasmota firmware, and [ESPHome](https://esphome.io/)/[Tasmota](https://tasmota.github.io/docs/) Integration

### Official Home Assistant Tuya Integration
If you plan on using the Official (Cloud Based) Tuya Integration in Home Assistant, you'll need to set up a Tuya IoT Platform Account (see link above for full instructions.) This is by far the easiest way to get your Tuya devices working with Home Assistant, but it also means that they are still reliant on a cloud service to work.

To set up this Integration, you'll need the following information:
- Tuya IoT Access ID
- Tuya IoT Access Secret
- Email & Password that you use to log in to the Smart Life or Tuya phone app
- Your Tuya device(s) added to one of these two apps

![Official Tuya Integration Setup Screen](https://i.imgur.com/kivCL4f.png)

### LocalTuya Home Assistant Tuya Integration
If you want a local-only option, there is a third-party Tuya Integration available for Home Assistant. It does require a bit more to set up, but frees you from the cloud. There are full instructions in the link above, but in a nutshell, you'll still need to set up a Tuya IoT Platform Account like the Official Tuya Integration and add your device(s) to the phone app. 

Additionally, you also need to know the following information about each device:
- Local Key
- Host ( Device IP Address on your local network)
- Device ID
- Protocol Version
- DP IDs (in most cases, the Local Tuya Integration will automatically retreive these IDs for you, but you'll still need to know which DP ID corresponds to which command; ie DP #1 turns your device on/off, DP #2 sets the temperature, etc.)

![Local Tuya Integration Setup Screen](https://i.imgur.com/1EhckPy.png)

### Flashing your own firmware with Tasmota or ESPHome
If you want to unlock the full potential of your device, and customize it however you see fit, both the previously linked Tasmota and ESPHome projects will allow you to do this. (For the purpose of this project, I'll be using ESPHome, but Tasmota is another option that will work.) Basically, you will replace the stock Tuya firmware running on your device, and add it to Home Assistant using the ESPHome Integration. While setting up a Tuya IoT Platform Account isn't strictly necessary for this setup, you will still need to know your device DP IDs, and which controls they are mapped to, so setting up a Tuya account does make this this process much easier by taking out all of the guesswork.

## Tuya Data
Like many Tuya IOT devices, this unit has one Tuya MCU to handle the actual machine operation, and a second Wifi MCU to handle wireless duties, communicating back and forth to the main MCU via serial, and to the Tuya cloud servers or your local Home Asssistant instance via Wifi. Since this device uses a secondary Tuya MCU, it is important that we map out the DP IDs before disassembly or flashing. Regardless of whether you plan on going with the LocalTuya or ESPHome/Tasmota options, you'll need to get some information first.

### Obtaining Local Key
The local_key is needed for local device access (ie, the Local Tuya Integration). If you're going to flash with ESPHome or use the cloud-based Home Assistant Integration, this isn't needed, and you can open the [API Explorer](#open-the-api-explorer), and jump down to [Obtaining the DP IDs](#obtaining-the-dp-ids). That said, it's a good idea to have either way in case anything goes wrong.

#### Open the API Explorer
Once you have your account setup and your devices added, select "Cloud" --> "Development" from the left menu:

![Development](https://i.imgur.com/hHxBilu.png)

From there, select your project, and you will be taken to the "Overview":

![Overview](https://i.imgur.com/bD8pIab.png)

Select the "Devices" tab at the far right:

![Devices](https://i.imgur.com/8RxniO2.png)

Record your "Device ID". You'll need it for the next steps. If you're using the Local Tuya Integration, you'll need to enter it when setting up the Integration in HA.

Next, navigate to "Cloud" --> "API Explorer":

![API Explorer Navigation](https://i.imgur.com/JiSNpJh.png)

This should open the API Explorer in a new window/tab:

![API Explorer Window](https://i.imgur.com/kH16TVv.png)

(If you're using ESPHome and don't need the Local Key, you can skip the next section and jump to [Obtaining the DP IDs](#obtaining-the-dp-ids))

#### Local Key
Navigate to "General Device Capabilities"-->"General Devices management" --> "Get Device Information", enter the "Device ID" you recorded in the previous step, and click the "Submit Request" button:

![Get Device Information](https://i.imgur.com/dua6Ome.png)

(Make sure you select the correct "Get Device Information" menu option! There are two; the one you want has each word capitalized. The lower case version will not give you all the information you need in the next step!)

This should return a JSON list of your Device Information Attributes. The one you're after is "local_key":

![Local Key](https://i.imgur.com/KE3bpDZ.png)

Copy this down. You'll need it when configuring the Local Tuya Integration.

### Obtaining the DP IDs
There are a pair of good guides on how to obtain the DP IDs [here](https://linkdhome.com/articles/local-tuya-device-control-in-homekit) and [here](https://www.zigbee2mqtt.io/advanced/support-new-devices/03_find_tuya_data_points.html). The first method is a bit eaiser, but I've found that it often misses some of the DP IDs.

Navigate to "Smart Home Device System" --> "Device Control" --> "Get Device Specification Attribute", enter the "Device ID" you recorded in the previous step, and click the "Submit Request" button:

![Get Device Specification Attribute](https://i.imgur.com/eNggJU9.png)

(Again, make sure you select the correct "Get Device Specification Attribute" menu option! There are two; the one you want has each word capitalized. The lower case version will not give you all the information you need in the next step!)

This should return a JSON list of your DP IDs, with a code/descritpion, data type, and valid value range for each DP:
```
{
  "result": {
    "category": "qn",
    "functions": [
      {
        "code": "switch",
        "dp_id": 1,
        "type": "Boolean",
        "values": "{}"
      },
      {
        "code": "temp_set",
        "dp_id": 2,
        "type": "Integer",
        "values": "{\"unit\":\"°C\",\"min\":4,\"max\":37,\"scale\":0,\"step\":1}"
      },
      {
        "code": "lock",
        "dp_id": 7,
        "type": "Boolean",
        "values": "{}"
      },
      {
        "code": "temp_unit_convert",
        "dp_id": 19,
        "type": "Enum",
        "values": "{\"range\":[\"c\",\"f\"]}"
      },
      {
        "code": "temp_set_f",
        "dp_id": 20,
        "type": "Integer",
        "values": "{\"unit\":\"℉\",\"min\":40,\"max\":99,\"scale\":0,\"step\":1}"
      }
    ],
    "status": [
      {
        "code": "switch",
        "dp_id": 1,
        "type": "Boolean",
        "values": "{}"
      },
      {
        "code": "temp_set",
        "dp_id": 2,
        "type": "Integer",
        "values": "{\"unit\":\"°C\",\"min\":4,\"max\":37,\"scale\":0,\"step\":1}"
      },
      {
        "code": "temp_current",
        "dp_id": 3,
        "type": "Integer",
        "values": "{\"unit\":\"°C\",\"min\":-9,\"max\":37,\"scale\":0,\"step\":1}"
      },
      {
        "code": "lock",
        "dp_id": 7,
        "type": "Boolean",
        "values": "{}"
      },
      {
        "code": "temp_unit_convert",
        "dp_id": 19,
        "type": "Enum",
        "values": "{\"range\":[\"c\",\"f\"]}"
      },
      {
        "code": "temp_set_f",
        "dp_id": 20,
        "type": "Integer",
        "values": "{\"unit\":\"℉\",\"min\":40,\"max\":99,\"scale\":0,\"step\":1}"
      },
      {
        "code": "temp_current_f",
        "dp_id": 21,
        "type": "Integer",
        "values": "{\"unit\":\"℉\",\"min\":16,\"max\":99,\"scale\":0,\"step\":1}"
      }
    ]
  },
  "success": true,
  "t": <redacted>,
  "tid": "<redacted>"
}
```

"Function" DP IDs are DPs you can control (turning the device on/off, setting the temperature, activating the child lock, etc.) These will be mapped to controllable entities in Home Assistant like switches, climate, etc.

"Status" DP IDs are non-controllable DPs, and will be mapped to sensor entities in Home Assistant.


From the JSON data, we now have the following DPs mapped:
|Description|DP Code|Data Type|DP ID|Values|
|:-|:-|:-|:-|:-|
|Switch|switch|Boolean|1|0=off,1=on|
|Set Temperature (°C)|temp\_set|Integer|2|Int range (4-37)|
|Current Temperature (°C)|temp\_current|Integer|3|Int range (-9-37)|
|Child Lock|lock|Boolean|7|0=on, 1=off|
|Temperature Scale|temp\_unit\_convert|Enum|19|0=C, 1=F|
|Set Temperature (°F)|temp\_set\_f|Integer|20|Int range (40-99)|
|Current Temperature (°F)|temp\_current\_f|Integer|21|Int range (16-99)|

As I noted earlier, this method tends to miss some of the DP IDs. If you follow second [link](https://www.zigbee2mqtt.io/advanced/support-new-devices/03_find_tuya_data_points.html) I posted at the beginning of this section, you can get the remaining DP IDs:
|Description|DP Code|Data Type|DP ID|Values|
|:-|:-|:-|:-|:-|
|Mode|mode|Enum|4|0=auto, 1=low, 2=high|
|Brightness|level|Enum|5|0=off, 1=10%, 2=50%, 3=100%|
|Fault Alarm|fault|Boolean|13|0=clear, 1=fault|
|Week Program (freestyle)|week\_program1|timing|16|\*See note below|

\* This sends a string with the following format:

```
"TYPE":"<CREATE / DELETED>", "id":<9 Digit ID#>, "time":"24h time", "dps":"<true=turn on/false=turn off>,<Temp in C>,<Temp in F>", "loops":"<1=run, 0=don’t run; SMTWRFS>", "status":"<opened / closed>", "Notification":false
```

As an example, the following string would turn on the heater every day at 0600h, and set it to 65\*F/18\*C:

```
"TYPE":"CREATE", "id":#########, "time":"06:00", "dps":"true,18,65", "loops":"1111111", "status":"opened", "Notification":false
```

I will be using this heater in conjunction with Home Assistant, so I will not need the standalone scheduling functionality, and will skip it for the sake of simplicity.

## Hardware and Disassembly
### Product Information
The product in question is [this](https://heatstorm.com/collections/wall-heaters/products/phoenix-wifi-smart-heater) Wifi-enabled 1500W infrared heater. Product manual can be found [here](https://cdn.shopify.com/s/files/1/1873/7325/files/HS-1500PHX-WIFI_Manual-compressed.pdf).

![Heat Storm Phoenix Heater](https://i.imgur.com/CaoB4I4.jpg)

### Disassembly
Disassembly is relatively straightforward and will only require a #2 Phillips Screwdriver.
To replace the Tuya Wifi module, you will need a soldering iron, flux, solder wick, solder, and either a hot air station, hotplate, or some semi-advanced soldering skills.

Start by flipping the unit over and removing the (12) screws securing the clamshell enclosure together:

![Rear Screw Locations](https://i.imgur.com/fss9pPR.jpg)

Ten of the screws are accessible from the back, and the last two are screwed up from the bottom:

![Bottom Screw Locations](https://i.imgur.com/mw0i43r.jpg)

Carefully separate the two case halves and lay them out flat, taking care not to overextend the two cables connecting the touch input screen to the control board:

![Case Opened](https://i.imgur.com/VI3YjrI.jpg)

The Wifi module is installed on the touch panel on the front panel, and secured with two screws:

![Wifi/Touch Input Board Location](https://i.imgur.com/E61hDrj.jpg)

Once the two screws are removed, the WBR3 Wifi module is clearly visible:

![Wifi/Touch Input Board](https://i.imgur.com/NrOuuPm.jpg)

Disconnect the two cables. The smaller 2P cable unplugs at the Wifi board. The larger 8P unplugs at the power board:

![Wifi/Touch Input Board Removed](https://i.imgur.com/mbE8Fls.jpg)

### Replacing the Tuya Wifi Module
#### Hardware
The [WBR3 Tuya Wifi module](https://developer.tuya.com/en/docs/iot/wbr3-module-datasheet?id=K9dujs2k5nriy) is not compatible with ESPHome or Tasmota, and must be replaced with an Espressif module. You can use an ESP8266 based module like the ESP-12E or ESP-12F (main difference is just an ugraded Wifi antenna design on the 12F), or an ESP32 based module like the ESP-C3-12F or ESP-12H. I used an [ESP-12F](https://www.amazon.com/dp/B07YYLQJGN) for mine and [this](https://www.amazon.com/dp/B0BG2FBNLJ) burner board to make flashing easier, but any of these options will work. If you do use an ESP32 based module, just make sure that you adjust your ESPHome yaml accordingly.

[Blakadder](https://blakadder.com/replace-tuya-esp12/) has a good guide on the steps involved to replace a Wifi module.

For a small board like this with very few through-hole components in the area, I think using a hotplate is the easiest way to desolder and remove the old Tuya module. You can get a nice commercial version like the [MHP30 on Amazon](https://www.amazon.com/gp/product/B08P5MLGYG) for ~$110, or on AliExpress for ~$20 cheaper. If you're on a budget, you can find a basic hotplate like [this](https://www.amazon.com/dp/B07W1ZZH8T) for under $15, or [build your own](https://github.com/AfterEarthLTD/Solder-Reflow-Plate) for ~$25.

Closeup of the WBR3 MCU:

![Closeup of the WBR3 MCU](https://i.imgur.com/bFnpjhT.jpg)

Tuya Wifi MCU Removed:

![Tuya Wifi MCU Removed](https://i.imgur.com/MZkHu4f.jpg)

Wifi Module Comparison:

![Wifi Module Comparison](https://i.imgur.com/G8kqAn0.jpg)

Flashing the new ESP-12F module with my ESPHome yaml:

![Flashing the new ESP-12F module with ESPHome](https://i.imgur.com/ZoWq5dB.jpg)

ESP-12F MCU installed:

![ESP-12F MCU installed](https://i.imgur.com/BufziUu.jpg)

#### Software: ESPHome
Once you have the Tuya module desoldered, you will need to flash the new Espressif module with your ESPHome yaml code before installing.

If you don't already have the ESPhome Integration installed in Home Assistant, you can install it following [these](https://esphome.io/guides/getting_started_hassio.html) steps. You'll also want to set up your secrets file

Once you have the Integration set up and running, open it up, and click the "New Device" button:

![ESPHome New Device](https://esphome.io/_images/dashboard_empty.png)

This will pop up with a "New Device" Window. Select "Continue":

![New Device Window](https://i.imgur.com/HPYwSRp.png)

Give your new device a name and click "Next":

![Create Configuration](https://i.imgur.com/JFs3ZDO.png)

Select your device type and click "Next":

![Select your device type](https://i.imgur.com/ECXSRj2.png)

I'm using an ESP8266 based ESP-12F, so I selected ESP8266, but make sure you select the correct device type according to the Espressif MCU you are using.

This will open up a "Configuration created!" message. Click on the "Encryption key" to copy it to the clipboard, and then click "Skip":

![Configuration created!](https://i.imgur.com/u8Gtmt6.png)

This will close the prompt and take you back to the main ESPHome Page. Find your newly created device, and click the "Edit" button:

![Edit](https://i.imgur.com/r9t5Wqd.png)

This will open your heater.yaml (or whatever you named your device) yaml configuration file:

![Default heater.yaml](https://i.imgur.com/CE711To.png)

Replace the contents of this file by pasting the contents of [heater.yaml](docs/heater.yaml) into this file:

![Pasted yaml](https://i.imgur.com/95tjsSr.png)

Click on "Save" then "Install" at the top right corner:

![Install](https://i.imgur.com/esrof32.png)

This will open a new prompt. Select "Manual Download"

![Manual Download](https://i.imgur.com/VEQBWT9.png)

In the version prompt, select "Legacy format":

![Legacy Format](https://i.imgur.com/41znbhE.png)

Save this to an easy to find location. You'll need this file to upload to your new ESP MCU.

Instead of hardcoding sensitive data like Wifi credentials, OTA passwords, and API Encryption Keys, I use the secrets file to 

#### Software: Uploading your YAML config
