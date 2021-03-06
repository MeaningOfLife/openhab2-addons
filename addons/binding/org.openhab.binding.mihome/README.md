# Xiaomi Mi Smart Home Binding

This binding allows your openHAB to communicate with the Xiaomi Smart Home Suite. 

In order to connect the Gateway, you need to install the MiHome app 
from the [Google Play](https://play.google.com/store/apps/details?id=com.xiaomi.smarthome) or [AppStore](https://itunes.apple.com/app/mi-home-xiaomi-for-your-smarthome/id957323480).

## Setup

* Install the binding
* Setup Gateway to be discoverable

    1. Add Gateway 2 to your WiFi Network
    1. Install Mi Home app from [Google Play](https://play.google.com/store/apps/details?id=com.xiaomi.smarthome) or [AppStore](https://itunes.apple.com/app/mi-home-xiaomi-for-your-smarthome/id957323480) (your phone may need to be changed to English language first)
    1. Set your region to Mainland China under Settings -> Locale (seems to be required)
    1. Update gateway (maybe multiple times)
    1. Enable developer mode:

        1. Select your Gateway in Mi Home
        1. Go to the "..." menu and click "About"
        1. Tap the version number "Version : 2.XX" at the bottom of the screen repeatedly until you enable developer mode
        1. You should now have 2 extra options listed in Chinese
        1. Choose the first new option (fourth position in the menu or the longer text in Chinese)
        1. Tap the toggle switch to enable LAN functions. Note down the developer key (something like: 91bg8zfkf9vd6uw7)
        1. Make sure you hit the OK button (to the right of the cancel button) to save your changes

* In openHAB you should now be able to discover the Xiaomi Gateway
* From now on you don't really need the app anymore - only if you're keen on updates or you want to add devices (see below), which also can be done without the app
* Enter the previously noted developer key in openHAB Paper UI -> Configuration -> Things -> Xiaomi Gateway -> Edit -> Developer Key. Save
    (This is required if you want to be able to send controls to the devices like the light of the gateway)
* Your sensors should be getting discovered by openHAB as you add and use them

## Connecting sub-devices (sensors) to the Gateway

There are two ways of connecting Xiaomi devices to the gateway:

* Online - within the MiHome App
* Offline - manual

    1. Click 3 times on the Gateway's button
    1. Gateway will flash in blue and you will hear female voice in Chinese
    1. Place the needle into the sensor and hold it for at least 3 seconds
    1. You'll hear confirmation message in Chinese 
    1. The device appears in openHAB thing Inbox

* If you don't want to hear the chinese voice every time, you can disable it by setting the volume to minimum in the MiHome App (same for the blinking light)
* The devices don't need an internet connection to be working after you have set up the developer mode BUT you won't be able to connect to them via App anymore

## Important information

The binding requires port `9898` to not be used by any other service on the system.

## Full Example

**xiaomi.items:**

```
// Xiaomi Gateway
Dimmer Gateway_Brightness <dimmablelight> { channel="mihome:gateway:<ID>:brightness" }
Color Gateway_Color <rgb> { channel="mihome:gateway:<ID>:color" }
Dimmer Gateway_ColorTemperature <heating> { channel="mihome:gateway:<ID>:colorTemperature" }

// Xiaomi Temperature and Humidity Sensor
Number HT_Temperature <temperature> { channel="mihome:sensor_ht:<ID>:temperature" }
Number HT_Humidity <humidity> { channel="mihome:sensor_ht:<ID>:humidity" }
Number HT_Battery <battery> { channel="mihome:sensor_ht:<ID>:voltage" }

// Xiaomi Motion Sensor
Switch MotionSensor_MotionStatus <motion>  { channel="mihome:sensor_motion:<ID>:motion" }
DateTime MotionSensor_LastMotion "[%1$tY-%1$tm-%1$td  %1$tH:%1$tM]" <clock-on> { channel="mihome:sensor_motion:<ID>:lastMotion" }
Number MotionSensor_Battery <battery> { channel="mihome:sensor_motion:<ID>:voltage" }

// Xiaomi Plug
Switch Plug_Switch <switch> { channel="mihome:sensor_plug:<ID>:powerOn" }
Number Plug_Voltage <poweroutlet> { channel="mihome:sensor_plug:<ID>:loadVoltage" }
Number Plug_Power <energy> { channel="mihome:sensor_plug:<ID>:loadPower" }
Number Plug_Consumption <line-incline> { channel="mihome:sensor_plug:<ID>:powerConsumed" }

// Xiaomi Window Switch
Contact WindowSwitch_Status <window>  { channel="mihome:sensor_magnet:<ID>:isOpen" }
Number WindowSwitch_Battery <battery> { channel="mihome:sensor_magnet:<ID>:voltage" }

// Xiaomi Cube - see "xiaomi.rules" for action triggers
Number Cube_RotationAngle { channel="mihome:sensor_cube:<ID>:rotationAngle" }
Number Cube_RotationTime { channel="mihome:sensor_cube:<ID>:rotationTime" }
Number Cube_Battery <battery> { channel="mihome:sensor_cube:<ID>:voltage" }

// Xiaomi Switch - see "xiaomi.rules" for action triggers
Number Switch_Battery <battery> { channel="mihome:sensor_switch:<ID>:voltage" }
```

**xiaomi.rules:**

```
rule "Xiaomi Switch"
when
    Channel "mihome:sensor_switch:<ID>:button" triggered
then
    var actionName = receivedEvent.getEvent()
    switch(actionName) {
        case "CLICK": {
            <ACTION>
        }
        case "DOUBLE_CLICK": {
            <ACTION>
        }
        case "LONG_CLICK_PRESS": {
            <ACTION>
        }
        case "LONG_CLICK_RELEASE": {
            <ACTION>
        }
    }
end

rule "Xiaomi Cube"
when
    Channel 'mihome:sensor_cube:<ID>:action' triggered
then
    var actionName = receivedEvent.getEvent()
    switch(actionName) {
        case "MOVE": {
            <ACTION>
        }
        case "ROTATE_RIGHT": {
            <ACTION>
        }
        case "ROTATE_LEFT": {
            <ACTION>
        }
        case "FLIP90": {
            <ACTION>
        }
        case "FLIP180": {
            <ACTION>
        }
        case "TAP_TWICE": {
            <ACTION>
        }
        case "SHAKE_AIR": {
            <ACTION>
        }
        case "FREE_FALL": {
            <ACTION>
        }
        case "ALERT": {
            <ACTION>
        }
    }
end

rule "Xiaomi Motion Sensor"
when
    Item MotionSensor_MotionStatus changed
then
    if (MotionSensor_MotionStatus.state == ON) {
        <ACTION>
    } else {
        <ACTION>
    }
end

rule "Xiaomi Window Switch"
when
    Item WindowSwitch_Status changed
then
    if (WindowSwitch_Status.state == OPEN) {
        <ACTION>
    } else {
        <ACTION>
    }
end

// This rule is applicable for every battery powered sensor device
rule "Xiaomi Motion Sensor Low Battery"
when
    Channel "mihome:sensor_motion:<ID>:batteryLevel" triggered LOW
then
    <ACTION>
end
```