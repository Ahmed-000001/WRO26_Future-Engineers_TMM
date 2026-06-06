# WRO26_Future-Engineers_TMM
WRO 2026 Future Engineers — Self-Driving Car (MATRIX Future Innovators Set)
Team Overview
This repository documents the design, development, and engineering process of our autonomous self-driving vehicle built for the WRO 2026 Future Engineers challenge. Our robot is engineered using the MATRIX Future Innovators Set (MAFI900 V2) as the primary hardware platform. The sensing array utilizes direct distance telemetry via highly precise time-of-flight MATRIX Laser Sensors V2 operating seamlessly within the native framework. 
Our current architecture implements four independent distance sensor nodes: the primary system array consisting of one laser sensor positioned squarely on the front bumper and one on the right flank, plus an added secondary reference array mounted at the left and rear frames. This provides full spatial mapping across all axes, enabling real-time wall tracking, absolute distance-based obstacle avoidance, and precise corner maneuvering without relying on camera feeds or computer vision modules. 
Table of Contents
1.	Mobility and Mechanical Design
2.	Power and Sensor Architecture
3.	Software Architecture and Obstacle Strategy
4.	Systems Thinking and Engineering Decisions
5.	How to Build and Run
6.	Team Photos and Videos
1. Mobility and Mechanical Design
Chassis and Structure
The robot is built using the structural components from the MATRIX Future Innovators Set (MAFI900 V2), which includes heavy-duty aluminum alloy beams, bracket plates, and plastic connector pieces. These components provide a rigid, lightweight frame that keeps the center of mass low and central—vital for maintaining optimal tire traction during fast cornering on the WRO track mats. 
The front wheels are custom replacements rather than the original set wheels. We made this decision after early testing revealed that the default MATRIX front wheels had a wide turning radius, making it difficult to navigate the competition's tight corridor configurations reliably. The replacement wheels have a smaller diameter, providing a significantly tighter steering response. 
Overall vehicle dimensions are strictly maintained within the competition limits of 300 × 200 × 300 mm. 

Drive System
Unlike earlier design iterations that relied on a single DC motor sharing a solid axle, the current vehicle configuration uses a dual-motor drive setup. Two independent high-torque MATRIX DC motors are wired directly to the controller's integrated output channels: Motor 3 (M3) handles the Right Wheel and Motor 4 (M4) handles the Left Wheel.
To maximize control over acceleration profiles and eliminate loose drifting momentum when cutting power, both drive channels have electronic braking explicitly enabled via software configuration (MiniR4.M3.setBrake(true) and MiniR4.M4.setBrake(true)).
This dual-actuator arrangement enables:
•	Balanced straight-line torque distribution.
•	Electronic drift manipulation (disengaging or halting one wheel entirely to allow the car to smoothly pivot around track barriers).
•	Rigid stabilization under heavy braking conditions.
Steering
A high-precision servo motor from the MATRIX set handles steering maneuvers via physical structural linkages. Connected to the RC3 PWM channel on the Mini R4 controller, the servo works in close synchronization with our dual-motor speed variations to change the front wheel angles.
The software utilizes a very tightly tuned steering baseline centered at 95° (SERVO_CENTER = 95). Standard drift adjustments around walls rely on highly restricted angular deviations—allowing the vehicle to correct its orientation safely without oversteering.
Torque and Speed Considerations
The system modulates the dual motors using continuous output power adjustments between standard thresholds:
•	Standard Straightaways: The vehicle runs at a steady baseline of -60 power simultaneously on both the left and right wheels to maintain maximum velocity down corridors.
•	Drift Maneuvers: When correcting course near boundaries, the software safely sets the inside motor power to 0, while boosting the outside motor to -80 to execute smooth, rapid heading updates without losing momentum.
•	Sharp Turn Pivot: During sharp corner turns, the inside wheel is locked completely to 0 while the opposite wheel runs at full throttle (-95 power) to whip the frame around tight turns instantly.

Mechanical Iterations
•	Version 1: Original MATRIX front wheels used. The turning circle was too wide for narrow lanes, causing the robot to strike the outer wall on sharp bends. 
•	Version 2: Replaced front wheels with smaller-diameter external wheels. Achieved a tighter turning radius, but the vehicle tracking code required refinement. 
•	Version 3: The physical structure surpassed 300 mm in height due to our temporary top-mounted camera mast. We dismantled this layout to keep our physical envelope low profile. 
•	Version 4: Encountered motor torque imbalances where the front wheels drifted to the right. Additionally, overlapping software code made resolving camera parameters unnecessarily complex. 
•	Version 5: Completely removed the camera module and video stabilization components. Designed two rigid, low-vibration mounting brackets for our two MATRIX Laser Sensors V2—placing one flush on the front bumper and one directly on the side rail. 
•	Version 6 (Current Build): Upgraded from a single-motor configuration to an independent Dual-Motor Drive Layout (M3 / M4) with active electronic gear braking. Expanded our distance sensing framework into a Four-Sensor Network by mounting an additional new sensor pair on the front bumper and left chassis rail, providing multi-angle positioning safeguards across all coordinate lanes.

2. Power and Sensor Architecture
Power System
The robot uses the MATRIX Mini R4 controller's onboard power distribution system, fed from the official rechargeable battery pack included in the MATRIX set. The battery safely powers both the core controller and the actuators through the built-in motor driver circuitry. 
The primary configuration logic sets up the system profile for a standard 2-cell lithium power supply via MiniR4.PWR.setBattCell(2). The four laser sensors are powered natively via the controller's high-speed I2C digital power rails. 
Estimated Current Draw:
•	MATRIX Mini R4 Controller: ~300 mA 
•	Dual High-Torque Motors (M3 + M4 combined load): ~1100 mA
•	Steering Servo Motor (Active Adjustments): ~400 mA 
•	Four Matrix Laser Sensors V2 Array: ~80 mA combined
•	Total Peak Draw: ~1.88 A. The MATRIX battery pack and internal power regulators handle this continuous load smoothly, eliminating the voltage sag resets caused by the older computer-vision configurations.
Sensors
Rather than utilizing sensitive computer vision which degrades under shifting venue lighting, our current architecture relies on laser-focused distance measurements: 
•	MATRIX Laser Sensor V2 (Front): Positioned at the dead-center of the front bumper and routed through high-speed expansion register I2C4. It fires time-of-flight distance pulses straight ahead to detect oncoming walls or obstacle pillars. 
•	Secondary Sensor Array (Front/Left additions): To safeguard tracking across opposite corridors, a new secondary laser sensor is paired alongside the front array, while another new laser sensor is mounted flush on the left rail. This prevents blind spots on the robot's left side during complex avoidance pathways.
Sensor Calibration & Initialization
Upon booting, the robot initiates the system architecture using the native library calls:
1.	MiniR4.begin() fires to map the onboard memory registers. 
2.	The front laser (MiniR4.I2C4.MXLaserV2.begin()) and right laser (MiniR4.I2C3.MXLaserV2.begin()) execute initialization sequences to establish standard data loops.
3.	The steering servo centers itself completely at 95° (MiniR4.RC3.setAngle(95)) and holds for 1000 milliseconds to establish a precise straight-line mechanical baseline.
Wiring Overview
•	Front Laser Sensor V2 $\rightarrow$ MATRIX Mini R4 Digital I2C Port 4
•	Right Laser Sensor V2 $\rightarrow$ MATRIX Mini R4 Digital I2C Port 3
•	Left & Front Secondary Lasers $\rightarrow$ MATRIX Mini R4 Digital I2C Ports 1 & 2
•	Steering Servo Motor $\rightarrow$ MATRIX Mini R4 Servo PWM Channel RC3
•	Right Drive Motor $\rightarrow$ MATRIX Mini R4 Motor Port M3
•	Left Drive Motor $\rightarrow$ MATRIX Mini R4 Motor Port M4




3. Software Architecture and Obstacle Strategy
Overview
All logic runs natively on the MATRIX Mini R4 controller. The software is written in clean, high-performance C++ (Arduino framework) rather than high-latency interpreted graphical blocks, ensuring that distance tracking updates process instantly without communication lag. 
Core State Logic & Wall Tracking Loop
The vehicle handles autonomous driving through a continuous real-time state loop. Rather than keeping track of complex color values, the robot handles lane centering by measuring its exact distance from the right boundary wall using MiniR4.I2C3.MXLaserV2.getDistance(). It continuously evaluates three specific distance zones:
C++
// 1. TOO CLOSE: Smooth Left Drift
if (rightDistance < 150) {
    MiniR4.RC3.setAngle(LIGHT_LEFT); // Set to 103 degrees (nudge left)
    MiniR4.M3.setPower(-80);         // Engage right wheel hard
    MiniR4.M4.setPower(0);           // Halt left wheel to let car drift left
}
// 2. TOO FAR: Smooth Right Drift
else if (rightDistance > 250) {
    MiniR4.RC3.setAngle(LIGHT_RIGHT); // Set to 87 degrees (nudge right)
    MiniR4.M3.setPower(0);            // Halt right wheel
    MiniR4.M4.setPower(-80);          // Engage left wheel hard to pull right
}
// 3. SAFE ZONE: Straight Drive
else {
    MiniR4.RC3.setAngle(SERVO_CENTER); // Hold straight at 95 degrees
    MiniR4.M3.setPower(-60);           // Balanced standard forward speed
    MiniR4.M4.setPower(-60);           
}
By shutting down the inside motor entirely during tracking errors, the vehicle lets its own physical physics pivot it smoothly back into position, keeping adjustments highly reliable.


Emergency Front Obstacle Strategy
When the vehicle encounters an immediate oncoming track wall or obstacle block, it triggers an over-riding sequence using data from the front distance sensor (I2C4):
1.	The Critical Threshold: If the front laser reads a distance of less than 300 mm, the vehicle immediately breaks out of standard lane tracking.
2.	Step 1 (Hard Electronic Braking): The software cuts motor commands to zero. The electronic gear brakes lock the axles for 200 milliseconds, protecting internal gears from sudden stress before direction changes.
3.	Step 2 (Straight Reversal): Both motor channels flip polarities (M3 and M4 set to +50 power), driving the vehicle straight back for 400 milliseconds to open up turning clearance.
4.	Step 3 (Pivot Swerve): The steering servo kicks to a sharp angle (103°), the left motor drops to 0, and the right motor pushes forward aggressively at -95 power for 500 milliseconds. This causes the car to spin sharply to the left on its own axis, clearing the obstacle before resetting back into the standard right-wall tracking routine.
4. Systems Thinking and Engineering Decisions
Key Design Decisions & Trade-offs
Migration from MicroPython/Python to Native C++
While early prototypes utilized Python environments for quick structural verification, our final software architecture migrated completely to native C++ using the Arduino IDE framework. Python interpreters introduce slight execution overhead. By writing low-level C++ scripts, our distance sensors can poll data at maximum processor speed with zero loop latency, ensuring the robot reacts immediately when moving fast down corridors.
Transition to Dual-Motor Pivot Configuration
Trading a single-motor axle for independent M3 and M4 wheel channels significantly improved our steering agility. In tight 600 mm competition lanes, mechanical steering limits can cause the front wheels to lock up. By using software to cut power to individual wheels, we can slip the chassis sideways around corners effortlessly.
Why we chose a Four-Sensor Laser Array over a Camera
Computer vision setups are prone to tracking failures due to overhead venue lighting glares and motor vibrations. By integrating four direct MATRIX Laser Sensors V2 (Front, Right, and the newly added Front-Left secondary configurations), the vehicle gains absolute, millimeter-precise spatial awareness. Distance sensing is unaffected by lighting changes, takes up less space, and reduces our script size to an elegant layout. 
Constraints Managed
•	Size Boundaries: Completely removing the bulky camera mast dropped our total vehicle height down to 145 mm, comfortably clear of the 300 mm height ceiling. 
•	Power Efficiency: Dropping the camera module reduced total logic rail current draw by ~350 mA, completely eliminating the voltage sags that caused controller resets during early testing phases.
Iteration Summary
•	v1: Initial MATRIX set build with default front wheels. 
•	v2: Swapped front wheels to custom, smaller-diameter tires to fix the wide turning radius. 
•	v3: Prototyped a camera mast, but it exceeded height constraints and suffered from severe vibration blur. 
•	v4: Experienced rightward tracking drift and code overlap trying to process camera scripts. 
•	v5: Removed camera entirely. Integrated dual MATRIX Laser Sensors V2 (Front and Left) for simplified distance navigation. 
•	v6 (Current Build): Implemented an independent Dual-Motor Drive Layout (M3/M4) with electronic braking. Migrated the primary tracking focus to the Right Wall, and expanded the hardware layout to include a Four-Sensor Network (adding extra front and left laser sensors) to provide full spatial awareness.
5. How to Build and Run
Requirements
•	Arduino IDE 2.x with the Arduino UNO R4 board manager package installed. 
•	Native MatrixMiniR4.h system libraries. 
•	Four MATRIX Laser Sensors V2 hooked up to your controller's I2C expansion registers. 
Upload Steps
1.	Open your code repository workspace on your computer. 
2.	Launch src/main/main.ino within the Arduino IDE. 
3.	Set your target board profile to Arduino UNO R4 WiFi. 
4.	Connect your MATRIX Mini R4 controller to your computer using a USB-C data cable. 
5.	Click Upload. Once completed, disconnect the cable and deploy the vehicle on the track mat using the official battery pack. 


