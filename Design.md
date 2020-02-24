# RDDF unit Design
## Campbell Maxwell

### Outline
I want to use a gyroscope, accelerometer and optical ToF to build a device to take readings and produce vectors which can be extrapolated into planes, which maintain their relative position and dimensions when additional planes are measured.

I will use an HM-10 module to communicate data to the android app which can will read the data and build a visual representation. Maybe send raw data to be processed to save power and time on the device. Either represent the constructed data in 3D or in multiple 2D planes.

### Hardware
I am considering using L3G4200D three-axis l2c/SPI digital gyroscope, LIS344ALH three-axis analogue output accelerometer, TF Mini ToF unit, HM-10, Arduino Mega controller board. Probably a laser pointer to position the device (being mindful of interference with the ToF), a push button to trigger the ToF to get a reading.

I would like to use two ToF units to get a vector with one button push, as well as having less calculations due to a constant angle and distance between vectors, but this can still be achieved with the unit requiring two presses to establish a vector, slightly more calculation but a lot less expense.

I may be able to exclude gyroscope if accelerometer can provide this funcionality with enough accuracy, as I am pretty sure it will provide absolute values due to having gravity as a reference point. Not sure if gyro has this also, if so would likely be worth including regardless of accelerometer. If I can find an accelerometer using 5v or a smaller range this would be ideal, as I will need to experiment first, but current choice may not be accurate enough (with arduino analogue detecting increments of 0.005v I will be able to detect changes of ~74mm/s2) 

### Translating Data
There are several factors and calculations needed to obtain useable vectors representing the dimensions of a wall.
#### ToF Vectors:
This depends on whether I use one or two ToF units.
Using two will be easier to calculate with less room for error, but much more expensive than with one.
With two, I think the best option is to have them mounted parallel to each other, so we have a known distance between them, and the length of each reading will provide us with the angle and direction of the vector.
Ideally data for each wall and roof and floor will be obtained which will give us the actual dimensions from wall to wall. The accelerometer comes into play here as we will be moving to take each reading, therefore this will be used to fill in blanks (as well as provide the relative position of each reading).
One ToF will require two readings for each wall. We will also have to move to take each reading so will need to use the accelerometer and the gyroscope to adjust for the movement. Should be able to translate this to connect to the start of the first vector and then use the current position + the distance to the end of each vector to create the connecting one and get the angle and direction of the wall.
What has not been mentioned yet is that most rooms are not cubic or rectanglar shaped, but have alcoves, or more than four walls. I need a way to adjust the drawing on the android to be able to add more measurements which contradict the original ones, and then recalculate the shape of the room accordingly.

#### Gyroscopic Vector Adjustment:
I need to be taking constant gyroscope readings to allow for a tilt of the device when taking a reading. For example a vertical angle when taking a measurement of a wall, as this would potentially add extra distance giving an erroneous reading. That example is not too difficult, as we get the angle relative to a perfectly flat horizontal angle of the device and just use trig to get the adjacent value.

#### Accelerometer Relativity:
This will be one of the core (and more complex) calculations, and will be ongoing to maintain a relative position of the device for the whole set of readings to obtain. The module has gravity as a reference point for angular readings, but if the device is switched off it would lose the reference point. This can be stored on the phone perhaps, would need a three axis grid position, relative to the current dimensions.
Also the accuracy of this module I am uncertain about so far. This will be VERY complex as I will need to start with v0 = 0m/s at the point the first ToF reading is made, then set a very small (less than 10ms) time interval to constantly obtain acceleration readings from the sensor and multiply them by the known time interval to get the instantaneous velocity value, then use the previous value as the v0 to get the actual value. This must also be done for each axis to provide us with the directional data for the velocity vector. Could try and do this for each change in output for acceleration, but would need a way to calculate the change in time.
V(current) = V(previous) + a*t   for each axis. Than using pythagoras on the values to get the final velocity vector and therefore the displacement of position.
A huge drawback would be that this process would have to be perpetual, even if the device is not moving. Will make fitting the rest of the program in between trickier if it has to grow. Also any errors are constantly growing.
I am currently considering not using the accelerometer and having the device start it's life as a floor mounted unit to avoid movement. This would still use the gyroscope, only need one ToF module and also use a combination of motors to rotate and raise the line of sight. It would likely start pointing down so we can use trig to obtain the height from the floor (unit could be mounted on a tripod, or on the floor directly without needing to know it's initial height offset. 
Height of device = ToF distance reading * Cos(vertical angle of ToF from pointing straight down)

### Arduino Build
First step would be to pair the android app with the device using BLE, can just wait for connection as there is no use allowing any functionality before this.
Assuming the ToF unit will start in a downwards pointing position to calculate the height offset.
Wait for user input from the android to determine next move - either automatic scan routine or a user controlled one.
Automatic routine will rotate taking multiple scans at set angles, then raise another set angle and repeat until we have the room and ceiling values. Could either wait until the end and transfer all data, or do it constantly.
User controlled will wait and respond to the user movement inputs and the take reading input. Can send data back to phone for imaging.

### Android Build
Probably start with attempting to pair with the device via BLE, or see if the user wants to look at existing plans and not pair immediately.
If the user is starting a scan then have a button to start which signals to the device to start the pre programmed scan.
Could maybe have an option to go to a user controlled scan, with a new view with joystick or touch pad or just multi button controls.
Need to obtain each calculated vector / plane from the device and translate it to a visual representation. Ideally have an option for 2D or 3D or maybe do both and update data as it arrives.
Be able to save a plan for later access.

### Final Notes
Although I mention the hand held device method using the accelerometer to maintain a known relative position, this will likely be a later upgrade as that functionality is clearly going to require a lot more research, testing, and complexity.
