
DIY CNC Fusion 360 CAM posts processor
====

This is modified fork of https://github.com/martindb/mpcnc_posts_processor

CAM posts processor for use with Fusion 360 and [MPCNC](https://www.v1engineering.com/assembly/) with RAMPS or any 3-axis DIY CNC.
Supported firmwares:
- Marlin 2.0
- Repetier firmware 1.0.3 (not tested. gcode is same as for Marlin)
- GRBL 1.1
- RepRap firmware (Duet3d) 

Some design points:
- Setup operation types: Milling, Water/Laser/Plasma
- Support mm and Inches units (**but all properties MUST be set in MM**)
- Rapids movements use seperate G0 moves, first to move in Z and then to move inXY. Moves use independent travel speeds for XY and Z.
- Arcs support on XY plane (Marlin/Repetier/RepRap) or all panes (Grbl)
- Tested with LCD display and SD card (built in tool change require printing from SD and LCD to restart)
- Support for 3 different laser power using "cutting modes" (through, etch, vaporize)
- Support 2 coolant channels. You may attach relays to control external devices - as example air jet valve.
- Customizable level of verbosity of comments
- Support line numbers
- Support GRBL laser mode (**be noted that you probably to have enabled laser mode [$32=1](https://github.com/gnea/grbl/wiki/Grbl-v1.1-Laser-Mode)**)

![screenshot](/screenshot.jpg "screenshot")

# User Properties

## Group 1: Job Properties
Use these properties to control overall aspects of the job.

|Title|Description|Default|
|---|---|---|
Job: Duet: Milling Mode|GCode command to setup Duet3d milling mode.|**"M453 P2 I0 R30000 F200"**|
Job: Duet: Laser Mode|GCode command to setup Duet3d laser mode.|**"M452 P2 I0 R255 F200"**|
Job: Goto 0 at end|Go X0 Y0 at gcode end. Useful to find if your machine loss steeps or have any other mechanic issue (like loose pulleys). Also useful for repetitive jobs. Only apply if not using gcodeStopFile.|**true**|
Job: Marlin: Manual Spindle On/Off|Set it to true when the motor of your spindle is controlled by manual switch. So the preprocessor will issue additional pauses for TURN ON/TURN OFF the motor.|**true**|
Job: Marlin: Enforce feedrate|Add feedrate to each movement g-code.|**false**|
Job: Separate words|Specifies that the words should be separated with a white space.|**true**|
Job: Line increment|Increment for sequence numbers.|**1**|
Job: Line start|First sequence number.|**10**|
Job: Line numbers|Show sequence numbers.|**false**|
Job: Reset on start (G92)|Set origin when gcode start (G92 X0 Y0 Z0). Only apply if not using
GcodeStartFile.|**true**|
Job: Use Arcs|Use G2/G3 g-codes for circular movements.|**true**|

Job: Firmware|Target firmware (marlin 2.0 or Repetir 1.0.3 / GRBL 1.1) / RepRap Firmware.|**Marlin**|

## Group 2: Feedrate Scaling
Use thes properties to scale the feedrate. Scaling will only occur if [Feed:Scaled feedrate] is true.

Scaling ensures that no G1 cut exceeds the speed capablities of the X, Y, or Z axes.
The cut's toolpath feedrate is projected onto the X, Y and Z axes. In turn each axis is tested
to see if its cut speed is within the limits of that axis. If not, then all axes feedrates are
scaled proportionatly to bring it within limits. This is repeated for all axes. The three axis
feedrates are then merged to create a new toolpath feedrate which is then limited to ensure it
doesn't exceed [Feed: Max toolpath speed].

Note: Because scaling considered 3 dimensional movement a resulting toolpath's feedrate may be
greater then the X, Y or Z limits. Consider a small movement in Z compared to much larger XY
movements. This will produce a toolpath feedrate that appears to exceed the limits of Z but in
reality this is appropriate because Z is traveling a much shorter distance then XY for the same
period of time.

|Title|Description|Default|
|---|---|---|
Feed: Travel Speed X/Y|High speed for travel movements X & Y (mm/min).|**2500 mm/min**|
Feed: Travel Speed Z|High speed for travel movements Z (mm/min).|**300 mm/min**|
Feed: Scaled feedrate|Scale cut feedrates to respect XY and Z max cut speeds.|**false**|
Feed: Max cut speed X or Y|Maximum cut speed along the X or Y axes (mm/min).|**900 mm/min**|
Feed: Max cut speed Z|Maximum cut speed along the Z axis (mm/min).|**180 mm/min**|
Feed: Max toolpath speed|Maximum cut speed along the toolpath (mm/min).|**10000 mm/min**|

# Group 3: Map G1->G0

Performs two actions by allowing G1 cuts to be mapped to G0 Rapid movements.

1. Resolves the loss of the initial positioning movement at the beginning of a cut toolpath (often identified in forums as the tool being initially dragged across the work surface).

2. Allows G1 XY cut movements that occur at a height >= [Feed: Safe Z for Rapids] to be converted to
Rapids. If [Feed: Allow Rapid Z] is true then it also converts G1 Z cut movements that either move upwards and end above the Safe Z, or move downwards with the start and end positions both above Safe Z. 

|Title|Description|Default|
|---|---|---|
Feed: G1 -> G0|Allow G1 cuts to be converted to Rapid G0 moves when safe and appropriate.|**false**|
Feed: Safe Z for Rapids|A G1 cut's Z must be >= to this to be mapped to a Rapid G0.|**10**|
Feed: Allow Rapid Z|Include vertical cut if they are safe.|**true**|

## Group 4: Tool change

|Title|Description|Default|
|---|---|---|
Change: Disable Z stepper|Disable Z stepper when change a tool|**false**|
Change: Enabled|Enable tool change code (bultin tool change requires LCD display)|**true**|
Change: X|X position for builtin tool change|**0**|
Change: Y|Y position for builtin tool change|**0**|
Change: Z|Z position for builtin tool change|**40**|
Change: Make Z Probe|Z probe after tool change|**true**|

## Group 5: Z Probe

|Title|Description|Default|
|---|---|---|
Probe: On job start|Execute probe gcode on job start|**true**|
Probe: Plate thickness|Plate thickness|**0.8**|
Probe: Use Home Z|Use G28 or G38 for probing|**true**|
Probe: G38 target|Probing up to Z position|**-10**|
Probe: G38 speed|Probing with speed|**30**|

## Group 6: Laser/Plasma related

|Title|Description|Default|Values|
|---|---|---|---|
Laser: On - Vaporize|Persent of power to turn on the laser/plasma cutter in vaporize mode|**100**||
Laser: On - Through|Persent of power to turn on the laser/plasma cutter in through mode|**80**||
Laser: On - Etch|Persent of power to turn on the laser/plasma cutter in etch mode|**40**||
Laser: Marlin mode|Marlin mode of the laser/plasma cutter ()|**M106**|M106 S{PWM}/M107 = 0; M3 O{PWM}/M5 = 1; M42 P{pin} S{PWM} = 2;|
Laser: Marlin pin|Marlin custom pin number for the laser/plasma cutter|**4**||
Laser: GRBL mode|GRBL mode of the laser/plasma cutter|**M4**|M4 S{PWM}/M5 dynamic power = 4; M3 S{PWM}/M5 static power = 3;|

## Group 7: Manage coolant control pins

|Title|Description|Default|Values|
|---|---|---|---|
Coolant: A Mode|Enable issuing g-codes for control Coolant channel A|**0**|off=0; flood=1; mist=2; throughTool=3; air=4; airThroughTool=5; suction=6; floodMist=7; floodThroughTool=8|
Coolant: A Marlin On command|GCode command to turn on Coolant channel A|**M42 P11 S255**||
Coolant: A Marlin Off command|Gcode command to turn off Coolant A|**M42 P11 S0**||
Coolant: A GRBL|GRBL g-codes for control Coolant channel A|**M7**|M7 flood = 7; M8 mist = 8|
Coolant: B Mode|Enable issuing g-codes for control Coolant channel B|**0**|off=0; flood=1; mist=2; throughTool=3; air=4; airThroughTool=5; suction=6; floodMist=7; floodThroughTool=8|
Coolant: B Marlin On command|GCode command to turn on Coolant channel B|**M42 P6 S255**||
Coolant: B Marlin Off command|Gcode command to turn off Coolant channel B|**M42 P6 S0**||
Coolant: B GRBL|GRBL g-codes for control Coolant channel B|**M8**|M7 flood = 7; M8 mist = 8|

## Group 8: Override behaviour by external files

|Title|Description|Default|
|---|---|---|
Extern: Start File|File with custom Gcode for header/start (in nc folder)||
Extern: Stop File|File with custom Gcode for footer/end (in nc folder)||
Extern: Tool File|File with custom Gcode for tool change (in nc folder)||
Extern: Probe File|File with custom Gcode for tool probe (in nc folder)||

## Group 9: Write comments into g-code

|Title|Description|Default|
|---|---|---|
Comment: Write Tools|Write table of used tools in job header|true|
Comment: Sections|Write header of every section|true|
Comment: Activities|Write comments which somehow helps to understand current piece of g-code|true|
Comment: Trace Commands|Write stringified commands called by CAM|true|
Comment: Trace Movements|Write stringified movements called by CAM|true|


# Sample of issued code blocks

## Gcode of milling with manually control spindel

```G-code
;Fusion 360 CAM 2.0.4860
; Posts processor: MPCNC_Mill_Laser.cps
; Gcode generated: Sunday, December 2, 2018 1:57:21 PM GMT
; Document: cam_testpp v5
; Setup: Setup1
; 
; Ranges table:
; X: Min=2.588 Max=36 Size=33.412
; Y: Min=2.588 Max=36 Size=33.412
; Z: Min=-1 Max=15 Size=16
; 
; Tools table:
; T1 D=3.175 CR=0 - ZMIN=-1 - flat end mill
; T2 D=1.5 CR=0 - ZMIN=-1 - flat end mill

; *** START begin ***
G90
G21
M84 S0
G92 X0 Y0 Z0
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; *** START end ***

; *** SECTION begin ***
;2D Contour1 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 2.588 - X Max: 49.412
; Y Min: 2.588 - Y Max: 49.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
M0 Turn ON spindle
; COMMAND_COOLANT_ON
M117 2D Contour1
G0 Z15
G0 X49.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y49.412 F300
G1 X2.588
G1 Y2.588
G1 X49.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
;2D Contour2 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 9.587 - X Max: 42.412
; Y Min: 9.587 - Y Max: 42.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; COMMAND_COOLANT_ON
M117 2D Contour2
G0 Z15 F300
G0 X42.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y42.412 F300
G1 X9.587
G1 Y9.587
G1 X42.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
; --- CHANGE TOOL begin ---
; COMMAND_COOLANT_OFF
M400
M300 S400 P2000
G0 Z50 F300
G0 X0 Y0 F2500
; COMMAND_STOP_SPINDLE
M0 Turn OFF spindle
M0 Put tool 2 - 1.5mm
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; --- CHANGE TOOL end ---
;Trace1 - Milling - Tool: 2 - 1.5mm flat end mill
; X Min: 16 - X Max: 36
; Y Min: 16 - Y Max: 36
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
M0 Turn ON spindle
; COMMAND_COOLANT_ON
M117 Trace1
G0 Z15
G0 X36 Y36 F2500
G0 Z4 F300
; MOVEMENT_LEAD_IN
G1 Z-1 F300
; MOVEMENT_CUTTING
G1 Y16
G1 X16
G1 Y36
G1 X36
; MOVEMENT_LEAD_OUT
G1 Z4
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** STOP begin ***
M400
; COMMAND_COOLANT_OFF
; COMMAND_STOP_SPINDLE
M0 Turn OFF spindle
G0 X0 Y0 F2500
M117 Job end
; *** STOP end ***
```

## Gcode of milling with spindel controlled by M3/M4/M5

```G-code
;Fusion 360 CAM 2.0.4860
; Posts processor: MPCNC_Mill_Laser.cps
; Gcode generated: Sunday, December 2, 2018 1:56:26 PM GMT
; Document: cam_testpp v5
; Setup: Setup1
; 
; Ranges table:
; X: Min=2.588 Max=36 Size=33.412
; Y: Min=2.588 Max=36 Size=33.412
; Z: Min=-1 Max=15 Size=16
; 
; Tools table:
; T1 D=3.175 CR=0 - ZMIN=-1 - flat end mill
; T2 D=1.5 CR=0 - ZMIN=-1 - flat end mill

; *** START begin ***
G90
G21
M84 S0
G92 X0 Y0 Z0
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; *** START end ***

; *** SECTION begin ***
;2D Contour1 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 2.588 - X Max: 49.412
; Y Min: 2.588 - Y Max: 49.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 21000
M3 S21000
; COMMAND_COOLANT_ON
M117 2D Contour1
G0 Z15
G0 X49.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y49.412 F300
G1 X2.588
G1 Y2.588
G1 X49.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
;2D Contour2 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 9.587 - X Max: 42.412
; Y Min: 9.587 - Y Max: 42.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 20000
M3 S20000
; COMMAND_COOLANT_ON
M117 2D Contour2
G0 Z15 F300
G0 X42.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y42.412 F300
G1 X9.587
G1 Y9.587
G1 X42.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
; --- CHANGE TOOL begin ---
; COMMAND_COOLANT_OFF
M400
M300 S400 P2000
G0 Z50 F300
G0 X0 Y0 F2500
; COMMAND_STOP_SPINDLE
M5
M0 Put tool 2 - 1.5mm
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; --- CHANGE TOOL end ---
;Trace1 - Milling - Tool: 2 - 1.5mm flat end mill
; X Min: 16 - X Max: 36
; Y Min: 16 - Y Max: 36
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 21000
M3 S21000
; COMMAND_COOLANT_ON
M117 Trace1
G0 Z15
G0 X36 Y36 F2500
G0 Z4 F300
; MOVEMENT_LEAD_IN
G1 Z-1 F300
; MOVEMENT_CUTTING
G1 Y16
G1 X16
G1 Y36
G1 X36
; MOVEMENT_LEAD_OUT
G1 Z4
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** STOP begin ***
M400
; COMMAND_COOLANT_OFF
; COMMAND_STOP_SPINDLE
M5
G0 X0 Y0 F2500
M117 Job end
; *** STOP end ***
```

## Gcode of milling with spindel controlled by M3/M4/M5 with using Coolants (both A and B channels)

```G-code
;Fusion 360 CAM 2.0.4860
; Posts processor: MPCNC_Mill_Laser.cps
; Gcode generated: Sunday, December 2, 2018 2:06:54 PM GMT
; Document: cam_testpp v5
; Setup: Setup1
; 
; Ranges table:
; X: Min=2.588 Max=36 Size=33.412
; Y: Min=2.588 Max=36 Size=33.412
; Z: Min=-1 Max=15 Size=16
; 
; Tools table:
; T1 D=3.175 CR=0 - ZMIN=-1 - flat end mill
; T2 D=1.5 CR=0 - ZMIN=-1 - flat end mill

; *** START begin ***
G90
G21
M84 S0
G92 X0 Y0 Z0
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; *** START end ***

; *** SECTION begin ***
;2D Contour1 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 2.588 - X Max: 49.412
; Y Min: 2.588 - Y Max: 49.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 21000
M3 S21000
; COMMAND_COOLANT_ON
; >>> Coolant A ON
M42 P11 S255
M117 2D Contour1
G0 Z15
G0 X49.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y49.412 F300
G1 X2.588
G1 Y2.588
G1 X49.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
;2D Contour2 - Milling - Tool: 1 - 1/8inch flat end mill
; X Min: 9.587 - X Max: 42.412
; Y Min: 9.587 - Y Max: 42.412
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 20000
M3 S20000
; COMMAND_COOLANT_ON
; >>> Coolant A OFF
M42 P11 S0
; >>> Coolant B ON
M42 P6 S255
M117 2D Contour2
G0 Z15 F300
G0 X42.412 Y26 F2500
G0 Z5 F300
; MOVEMENT_PLUNGE
G1 Z1 F100
G1 Z-1
; 14
G1 Y42.412 F300
G1 X9.587
G1 Y9.587
G1 X42.412
G1 Y26
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** SECTION begin ***
; --- CHANGE TOOL begin ---
M400
G0 Z50 F300
G0 X0 Y0 F2500
; COMMAND_COOLANT_OFF
; >>> Coolant B OFF
M42 P6 S0
; COMMAND_STOP_SPINDLE
M5
M300 S400 P2000
M0 Put tool 2 - 1.5mm
; COMMAND_TOOL_MEASURE
; --- PROBE TOOL begin ---
M0 Attach ZProbe
G28 Z
G92 Z0.8
G0 Z50 F300
M0 Detach ZProbe
; --- PROBE TOOL end ---
; --- CHANGE TOOL end ---
;Trace1 - Milling - Tool: 2 - 1.5mm flat end mill
; X Min: 16 - X Max: 36
; Y Min: 16 - Y Max: 36
; Z Min: -1 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_CLOCKWISE
; >>> Spindle Speed 21000
M3 S21000
; COMMAND_COOLANT_ON
M117 Trace1
G0 Z15
G0 X36 Y36 F2500
G0 Z4 F300
; MOVEMENT_LEAD_IN
G1 Z-1 F300
; MOVEMENT_CUTTING
G1 Y16
G1 X16
G1 Y36
G1 X36
; MOVEMENT_LEAD_OUT
G1 Z4
; MOVEMENT_RAPID
G0 Z15
; *** SECTION end ***

; *** STOP begin ***
M400
; COMMAND_COOLANT_OFF
; COMMAND_STOP_SPINDLE
M5
G0 X0 Y0 F2500
M117 Job end
; *** STOP end ***
```

## Gcode of laser cutting

```G-code
;Fusion 360 CAM 2.0.4860
; Posts processor: MPCNC_Mill_Laser.cps
; Gcode generated: Sunday, December 2, 2018 2:07:32 PM GMT
; Document: cam_testpp v5
; Setup: Setup2
; 
; Ranges table:
; X: Min=-25 Max=25 Size=50
; Y: Min=-25.5 Max=25 Size=50.5
; Z: Min=0 Max=15 Size=15
; 
; Tools table:
; T1 D=0 CR=0 - ZMIN=0 - laser cutter

; *** START begin ***
G90
G21
M84 S0
G92 X0 Y0 Z0
; COMMAND_TOOL_MEASURE
; *** START end ***

; *** SECTION begin ***
;2D Profile1 - Laser/Plasma - Cutting mode: auto
; X Min: -25 - X Max: 25
; Y Min: -25.5 - Y Max: 25
; Z Min: 0 - Z Max: 15
M400
; COMMAND_START_SPINDLE
; COMMAND_SPINDLE_COUNTERCLOCKWISE
; COMMAND_COOLANT_ON
M117 2D Profile1
G0 Z15 F300
G0 X-9.94 Y-10.5 F2500
G0 Z0 F300
; >>> LASER Power ON
M106 S200
; COMMAND_POWER_ON
; MOVEMENT_LEAD_IN
G1 Y-10 F1000
G1 X-9.95
; MOVEMENT_CUTTING
G1 X-10
G1 Y10
G1 X10
G1 Y-10
G1 X-9.95
; MOVEMENT_LEAD_OUT
G1 X-9.96
G1 Y-10.5
; >>> LASER Power OFF
M107
; COMMAND_POWER_OFF
; MOVEMENT_RAPID
G0 Z5 F300
G0 X-9.99 Y-25.5 F2500
G0 Z0 F300
; >>> LASER Power ON
M106 S200
; COMMAND_POWER_ON
; MOVEMENT_LEAD_IN
G1 Y-25 F1000
G1 X-10
; MOVEMENT_CUTTING
G1 X-25
G1 Y25
G1 X25
G1 Y-25
G1 X-10
; MOVEMENT_LEAD_OUT
G1 X-10.01
G1 Y-25.5
; >>> LASER Power OFF
M107
; COMMAND_POWER_OFF
; MOVEMENT_RAPID
G0 Z15 F300
; *** SECTION end ***

; *** STOP begin ***
M400
; COMMAND_COOLANT_OFF
; COMMAND_STOP_SPINDLE
G0 X0 Y0 F2500
M117 Job end
; *** STOP end ***
```

# Resorces

[Marlin G-codes](http://marlinfw.org/meta/gcode/)

[PostProcessor Class Reference](https://cam.autodesk.com/posts/reference/classPostProcessor.html)

[Post Processor Training Guide (PDF document)](https://cam.autodesk.com/posts/posts/guides/Post%20Processor%20Training%20Guide.pdf)

[Enhancements to the post processor property definitions](https://forums.autodesk.com/t5/hsm-post-processor-forum/enhancements-to-the-post-processor-property-definitions/td-p/7325350)

[Dumper PostProcessor](https://cam.autodesk.com/hsmposts?p=dump)

[Library of exist post processors](https://cam.autodesk.com/hsmposts)

[Post processors forum](https://forums.autodesk.com/t5/hsm-post-processor-forum/bd-p/218)

[How to set up a 4/5 axis machine configuration](https://forums.autodesk.com/t5/hsm-post-processor-forum/how-to-set-up-a-4-5-axis-machine-configuration/td-p/6488176)

[Beginners Guide to Editing Post Processors in Fusion 360! FF121 (Youtube video)](https://www.youtube.com/watch?v=5EodQIY25tU)
