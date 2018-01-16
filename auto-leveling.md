# Hypercube Bed Leveling Process

At the time of last updating this document, Marlin firmware RCBugFix branch used was [commit](https://github.com/MarlinFirmware/Marlin/tree/8c07ac7f7c6da6d0a7e60581019c0b1e62732bf3)

I used Repetier Host to create the current edition of this document.

**Overview**

  - [Setup the bed](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#setup-the-bed)
  - [Setup the probe](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#setup-the-probe)
    - [Attach the proble](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#attach-probe)
    - [Calculate distance between probe tip and nozzle tip](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#calculate-distance-between-probe-tip-and-nozzle-tip)
  - [Generate auto level mesh](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#auto-level-the-bed-using-auto_bed_leveling_bilinear-5x5-grid)
  - [Print a calibration model](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#print-a-calibration-model)
  - [Troubleshooting](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#troubleshooting)
    - [Leveling the bed](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#leveling-the-bed)
    - [Check auto leveling is working](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#check-auto-leveling-is-working)
    - [Check probe repeatability](https://github.com/pflannery/Hypercube/blob/master/auto-leveling.md#check-the-probes-repeatability)

## Setup the bed

Usually done after moving the printer

- Give all the springs middle tension. My starting preference is 20mm between bed and lowest part of the spring.<br>
  I use a steel ruler to align each spring.

## Setup the probe

[LJ12A3-4-Z/BY PNP DC6-36V Inductive Proximity Sensor](http://www.banggood.com/LJ12A3-4-ZBY-PNP-DC6-36V-Inductive-Proximity-Sensor-Detection-Switch-p-982679.html?rmmds=myorder)

Some work at 5V but from my experience it's best to run them according to their spec levels. 
You can use a voltage divider to limit a higher voltage down to 5v if need be.

##### Attach Probe

- Put hotend nozzle flat to bed
- Use a 1mm object to slide under the probe and adjust the probe to be touching the top of the object (I use a steel ruler). Make sure that when you remove the object under the probe that the probe drop down to the bed when you pull away
- Tighten probe nuts with spanners or pliers if your using the original probe mount.<br>
  I use this [part](http://www.thingiverse.com/thing:2179807) because I find easier to align the probe and allows me to hand tighten the nuts because it's clipped in already. 

##### Calculate distance between probe tip and nozzle tip

1. Heat the nozzle and bed for material temperature (Just be mindful of any filament getting in the way when leveling, I tend to pull it out of the hotend while I do this.)
1. `M851 Z0` set the probe z offset to 0.
1. `G28` home all axes. [G28](http://reprap.org/wiki/G-code#G28:_Move_to_Origin_.28Home.29)
1. Move the nozzle to the centre of the bed using a control or [G0 or G1](http://marlinfw.org/docs/gcode/G000-G001.html).
1. Move the nozzle down until it's almost touching the bed using a feeler gauge like [this](http://www.ebay.co.uk/itm/26-BLADE-FEELER-GAUGE-SET-GUITAR-NECK-RELIEF-STRING-HEIGHT-LUTHIER-TOOL-GUAGE-/162403994221?hash=item25d0084a6d:g:0QMAAOSw54xUW2mG) or paper.<br> Knowing your gauge height will help with your initial print layer height (See calibration print settings below).The paper I've used in the past was around 0.1mm.
1. Bring the nozzle down until you can't slide the feeler gauge under the nozzle. Now raise the nozzle until the feeler gauge can be slid in and out underneath the nozzle without being caught by it.
1. `M114` and take note of the current z position, it can sometimes be below 0 (Read the first "Z:##" outputted in the logged result).
1. `M851 z%YOUR CALCULATED Z OFFSET%`. [M851](http://reprap.org/wiki/G-code#M851:_Set_Z-Probe_Offset)
1. `G28` home all axes. [G28](http://reprap.org/wiki/G-code#G28:_Move_to_Origin_.28Home.29)
1. Move nozzle to Z0 and double check with paper that it's the same. If not go back to step 5.
1. If all ok run `M500` to save settings

## Auto level the bed (using AUTO_BED_LEVELING_BILINEAR, 4x4 grid)

**Heat the hotend and bed before doing this**

  - `G28` home all axes. [G28](http://reprap.org/wiki/G-code#G28:_Move_to_Origin_.28Home.29)
  - `G29` run auto bed level. [G29](http://reprap.org/wiki/G-code#G29:_Detailed_Z-Probe)
  - `M500` to save these settings if you have EEPROM enabled in your configuration.h.<br>
    If you dont use EEPROM then you will need to auto level each time you restart your printer [M500](http://reprap.org/wiki/G-code#M500:_Store_parameters_in_EEPROM)
  - `M420 S1` needs to be run just before each print (i.e. last line of your start.gcode). [See discussion here for more info](https://github.com/MarlinFirmware/Marlin/issues/5996#issuecomment-287380079)

**Bare in mind that after you have created mesh level data with G29 and you ever change the z offset using M851 Marlin will now automatically update the mesh data.**

##### Print a calibration model

I use a 40mm cube from here http://www.thingiverse.com/thing:56671

|Property|Value|Notes|
|--------|-----|-----|
|Line Width|0.4mm|For a 0.4mm nozzle|
|Initial Layer Height|0.3mm|This should be no more than NozzleSize - GaugeHeight.<br>Using a gauge height of 0.1mm seems to work best with a 0.3mm initial layer height|
|Layer Height|0.2mm|Check out the [Prusa calculator](http://www.prusaprinters.org/calculator#layerheight) for your layer height settings|
|Top\Bottom Layer Count|2|Layer height * Top\Bottom Layer Count = 0.4mm|
|Wall Line Count|1|Line Width * Wall Line Count = 0.4mm|
|Infill|0%||
|Supports|No||
|Print Speed|30mm/s||
|Skirt Line Count|1||
|Skirt Distance|8mm||

Examine the first layer and tweek the z offset with M851. Increments of 0.2-0.4 usually works for me. But it will depend on the print result. 

At worst change bed springs but I've found I dont need this with auto leveling.

### Troubleshooting

##### Leveling the bed

  - Use the auto level data produced by the probe and adjust the springs to help level your bed if the report shows a large difference. 

    My bed is flat but I usually find a 1-2mm difference between the back and the front of the bed. I raise the back upwards (anti-clockwise turning) to close this gap. I then run the auto leveling procedure again. Repeat if I need to until I've got it around 0.5-0.8mm difference between the biggest corner gaps.
    
##### Check auto leveling is working

- At printer start-up you should see 
  ```
  Auto Bed Leveling:
  echo:  M420 S0
  ```
  If not then check you've enabled AUTO_BED_LEVELING_*** in your configuration.h [Example](https://github.com/pflannery/Hypercube/blob/master/configuration.h#L812)
  
- Check your leveling data has persisted using `M420 V`. This will show that the bed leveling data exists [M420](http://reprap.org/wiki/G-code#M420:_Enable.2FDisable_Mesh_Leveling_.28Marlin.29)

- Ensure you have `M420 S1` in your start.gcode just before your printer starts to print so to ensure that auto leveling is enabled. [Example](https://github.com/pflannery/Hypercube/blob/master/start.gcode#L18)
  
##### Check the probe's repeatability.

- First you need to uncomment `#define Z_MIN_PROBE_REPEATABILITY_TEST` in your configuration.h and reflash your firmware
- `G28` home all axes. [G28](http://reprap.org/wiki/G-code#G28:_Move_to_Origin_.28Home.29)
- `M48 P10 X100 Y100 V2 E l2` [M48](http://reprap.org/wiki/G-code#M48:_Measure_Z-Probe_repeatability)
- Repeat 2 to 3 times to see how much difference in the "Standard Deviation" your probe is reporting.
- If it's always far away from each test check the tightness of the probe mount.
  - Make sure the cable has a bit of slack above the probe because the cable bundle may be able to pull the probe around forcing it to wiggle out of position.
  - Make sure associated printer parts are also tight and not wiggling from cables.
  
Repeatability comparision

|Fixed z-endstop| | | | |
|-|-|-|-|-|
|Mean: -0.004750 |Min: -0.007 |Max: -0.003 |Range: 0.005|Standard Deviation: 0.001750|
|Mean: -0.009500| Min: -0.010 |Max: -0.007 |Range: 0.003|Standard Deviation: 0.001000|
|Mean: -0.010750 |Min: -0.012 |Max: -0.007 |Range: 0.005|Standard Deviation: 0.001601|

|Probe          | | | | |
|-|-|-|-|-|
|Mean: 0.433250 |Min: 0.422 |Max: 0.445 |Range: 0.023|Standard Deviation: 0.007163|
|Mean: 0.366750 |Min: 0.365 |Max: 0.367 |Range: 0.002|Standard Deviation: 0.001146|
|Mean: 0.369500 |Min: 0.365 |Max: 0.373 |Range: 0.008|Standard Deviation: 0.002179|

Example output from marlin log

```
M48 Z-Probe Repeatability Test
1 of 10: z: 0.370
2 of 10: z: 0.365
3 of 10: z: 0.370
4 of 10: z: 0.370
5 of 10: z: 0.367
6 of 10: z: 0.370
7 of 10: z: 0.370
8 of 10: z: 0.367
9 of 10: z: 0.373
10 of 10: z: 0.373
Finished!
Mean: 0.369500 Min: 0.365 Max: 0.373 Range: 0.008
Standard Deviation: 0.002179
```
