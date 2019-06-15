# homebridge-ring-alarm
 
[![CircleCI](https://circleci.com/gh/dgreif/ring-alarm.svg?style=svg)](https://circleci.com/gh/dgreif/ring-alarm)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=HD9ZPB34FY428&currency_code=USD&source=url)
 
This [Homebridge](https://github.com/nfarina/homebridge) plugin provides a platform for
[Ring Doorbells](https://shop.ring.com/pages/doorbell-cameras),
[Ring Cameras](https://shop.ring.com/pages/security-cameras),
the [Ring Alarm System](https://shop.ring.com/pages/security-system),
[Ring Smart Lighting](https://shop.ring.com/pages/smart-lighting),
and third party devices that connect to the Ring Alarm System.
 
 ## Installation
 
 Assuming a global installation of `homebridge`:
 
 `npm i -g homebridge-ring-alarm`
 
 ## Homebridge Configuration
 
 Add the `RingAlarm` platform in your homebridge `config.json` file.
 
 ```js
{
  "platforms": [
    {
      "platform": "RingAlarm",
      "email": "some.one@website.com",
      "password": "abc123!#",

      // OPTIONAL. See below for details
      "locationIds": ["488e4800-fcde-4493-969b-d1a06f683102", "4bbed7a7-06df-4f18-b3af-291c89854d60"],
      "alarmOnEntryDelay": false,
      "beamDurationSeconds": 60,
      "hideLightGroups": true
    }
  ]
}
```

### Optional Parameters

`locationIds`: Use this option if you only want a subset of your locations to appear in HomeKit. If this option is not included, 
all of your locations will be added to HomeKit (which is what most users will want to do).  

`alarmOnEntryDelay`: if `true`, HomeKit will register a delayed entry event as a triggered alarm.  
There are some households where this is a nice feature as a heads up if you have other people who
enter your house and you want an alert so that you can disable the alarm for them before it actually goes off.
This works well if you automatically arm/disarm on leave/arrive (see setup instructions below)

`beamDurationSeconds`: Ring smart lighting has a default behavior of only staying on for 60 seconds
when you turn on a light via the Ring app.  To force a duration when the light is turned on from HomeKit,
set this option to a specific number of seconds.
If this option is not set, the lights will use the duration from the previous time the light was turned on in the Ring app.
For light groups, this will default to 60 seconds.
The maximum value is `32767`, which is ~9.1 hours.

`hideLightGroups`: Ring smart lighting allows you to create lighting groups within the Ring app.
These groups are convenient for detecting motion in an area of your yard and turning on/off all lights
in the group.  However, you may wish to group the lights differently in HomeKit and ignore the 
groups you have configured in Ring.  If this option is `true`, your Ring groups (and their associated motion sensor)
will be ignored and will not show up in HomeKit.

`cameraStatusPollingSeconds`: How frequently to poll for updates to your cameras.  Information like 
light/siren status do not update in real time and need to be requested periodically.  Defaults to `30`

`cameraDingsPollingSeconds`: How frequently to poll for new events from your cameras.  These include motion and
doorbell presses.  Defaults to `5`

### Supported Devices
  * Cameras (Experimental)
    * Light On/Off
    * Siren On/Off
    * Motion - Requires motion alerts to be on for the camera in the Ring App.  If you have
    motion snooze or a motion schedule enabled, you will not receive motion events via HomeKit either.
    * Doorbell presses - Requires ring alerts to be on for the camera in the Ring App.
    * TODO: Battery
    * TODO: Snapshots and live streams
  * Security Panel
    * This is a software device that represents the alarm for a Ring location
    * Arm Home / Arm Away / Disarm alarm for Ring location.
    * Detect active burglar alarm
    * Sound/silence siren (shows as a "Ring Alarm Siren" switch)
  * Base Station
    * Set Volume (Not currently supported in Home, but works in other apps like Eve)
    * Battery status
  * Keypad
    * Set Volume (Not currently supported in Home, but works in other apps like Eve)
    * Battery status
  * Contact Sensor
    * Detect if sensor is open or closed
    * Tamper status
    * Battery status
  * Motion Sensor
    * Detect motion
    * Tamper status
    * Battery status
  * Ring Smart Lights (Motion Detector, Flood/Path/Spot Lights, Transformer)
    * On/Off
    * Brightness Level
    * Detect motion
    * Battery Status
  * Smoke Alarm
  * Carbon Monoxide Alarm
  * Smoke/Carbon Monoxide Listener
  * Smart Locks
  * Lights/Switches (Implemented but not tested)
    * On/Off
    * Brightness Level
    * Hue/Sat/Color Temp are _possible_, but currently not supported.
      Please open an issue if you have a device that you would be able to
      test these on.

### Alarm Modes

Ring Mode | HomeKit Mode
--- | ---
Disarmed | Off
Home | Home
Away | Away
Home | Night

Entry delays and bypassed sensors (ex. for Home mode) are all controlled in the Ring app.
These settings will automatically be used by HomeKit.

**Note**: Using `Night` mode in HomeKit will activate `Home` mode on the Ring alarm.
HomeKit should immediately switch to `Home` to match.  

### Changes Modes on Arrive/Leave Home

The Home app allows you to create an automation that runs when you arrive/leave home.  If you include an accessory or
scene that "allows access" to your home (e.g. Security System or Lock), the Home app will require you to manually verify
that you actually want to run the automation every time it gets triggered.  For anyone who wants to arm/disarm their
Ring Alarm automatically, this manual verification can be quite annoying.  To work around this issue, you can add two
"dummy" switches to you homebridge setup using [homebridge-dummy](https://www.npmjs.com/package/homebridge-dummy).  You
can then use these dummy switches to trigger your other automation (e.g. Arm/Disarm Ring Alarm).

#### Example Dummy Config
```json
"accessories": [
    {
      "accessory": "DummySwitch",
      "name": "Arrived Home"
    },
    {
      "accessory": "DummySwitch",
      "name": "Left Home"
    }
]
```

#### Example Home Automations

Trigger | Action
--- | ---
People Arrive | Turn on "Arrived Home"
People Leave | Turn on "Left Home"
"Arrived Home" turns on | Set Ring Alarm mode to Off
"Left Home" turns on | Set Ring Alarm mode to Away