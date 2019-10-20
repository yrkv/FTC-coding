### Preface

This document serves as a general sequence of "things to do" as a programmer for an FTC team. It should be used as a guide for what to focus on. It is not a definitive explanation for everything, but rather a semi-ordered collection of what I think is fairly critical for an FTC team to be able to use so that the software team does not hinder the team overall.

This is not meant to teach programming overall, but it's not so advanced that someone who is new to programming definitely cannot figure it out. Everything here is intended to be done using Java through either OnBot Java or Android Studio. 

Most of the time, examples used here will not be actually usable as FTC programs because I stripped away some of the necessary parts to make it shorter. However, the parts I am referring to can be copy/pasted into a functioning program and modified to fit your code.

This is essentially an abridged version of the [ftc_app wiki](https://github.com/ftctechnh/ftc_app/wiki) which assumes you can figure out how to get things to work. The main difference is that I fill in the gap in the wiki; the wiki does not properly explain the relatively high-level details of how the code should work. The reason I belive something like this is necessary is that the majority of the ftc_app wiki covers getting it to run rather than how everything in the code works.

## Introduction to FTC code

#### OpMode

Every program in FTC extends the `OpMode` class. When you initialize a program, the robot calls the `OpMode.init()` method. After that, it repeatedly calls `OpMode.init_loop()` until the start button is pressed. The rest of the methods follow intuitively from that. Your job as a programmer is to override those methods to do whatever it is that the robot should do at that point. Here is a list of some of the available user defined methods:

- `void init()`
    - This method will be called once when the INIT button is pressed.
- `void init_loop()`
    - This method will be called repeatedly when the INIT button is pressed. 
    - This method is optional. By default this method takes no action.
- `void start()`
    - This method will be called once when the PLAY button is first pressed.
    - This method is optional. By default this method takes no action.
- `void loop()`
    - This method will be called repeatedly in a loop while this op mode is running.

#### LinearOpMode

`LinearOpMode` serves as an optional extension/replacement to `OpMode` which lends itself well to autonomous code. Because of this, I will be using it for the autonomous section for the sake of simplicity and brevity. Instead of many methods with different trigger events, it has a single `LinearOpMode.runOpMode()` method. It is essentially the same as the `OpMode.init()` method. To get the same behavior as the other methods in `OpMode`, one can use while loops to repeat sections of code and use the following methods to control the flow:

- `void waitForStart()`
    - Pauses the Linear Op Mode until start has been pressed or until the current thread is interrupted.
- `void sleep(long milliseconds)`
    - Sleeps for the given amount of milliseconds, or until the thread is interrupted.
- `boolean opModeIsActive()`
    - Answer as to whether this opMode is active and the robot should continue onwards. If the opMode is not active, the OpMode should terminate at its earliest convenience.

#### Using devices

Any device (such as a motor, servo, or sensor) should be declared at the class level as an instance field so that it can be used in the entire program. For any common sensor, there is a sample which demonstrates its usage.

In `init()`, you should initialize the variable by obtaining the hardware object. After that, make whatever configuration settings you might need (such as reversing the drive motors on one side). 

Then in `loop()`, you can actually use the device. In the case of a motor, this would be setting the power to a value. For a servo, it would be setting the position. For a sensor, it would be reading a value from it.

```java
// This is the sample `BasicOpMode_Iterative` with only the relevant parts
public class BasicOpMode_Iterative extends OpMode
{
    // declare devices
    private DcMotor leftDrive;
    private DcMotor rightDrive;
    private Servo arm;

    public void init() {
        // Initialize the hardware variables. Note that the strings used here as parameters
        // to 'get' must correspond to the names assigned during the robot configuration
        // step (using the FTC Robot Controller app on the phone).
        leftDrive  = hardwareMap.get(DcMotor.class, "left_drive");
        rightDrive = hardwareMap.get(DcMotor.class, "right_drive");
        arm = hardwareMap.get(Servo.class, "arm_servo");

        // Most robots need the motor on one side to be reversed to drive forward
        // Reverse the motor that runs backwards when connected directly to the battery
        leftDrive.setDirection(DcMotor.Direction.FORWARD);
        rightDrive.setDirection(DcMotor.Direction.REVERSE);
    }

    public void loop() {
        // Tank Mode uses one stick to control each wheel.
        // - This requires no math, but it is hard to drive forward slowly and keep straight.
        double leftPower  = -gamepad1.left_stick_y;
        double rightPower = -gamepad1.right_stick_y;

        // Send calculated power to wheels
        leftDrive.setPower(leftPower);
        rightDrive.setPower(rightPower);

        // Set servo position to gamepad2.left_stick_y
        arm.setPosition(gamepad2.left_stick_y);
        // The sticks have a range of [-1, 1] while servos accept [0, 1].
        // This would need to be adjusted in a real program.
    }
}
```

#### Telemetry

Sometimes the robot might not be working the way you expect. An easy way to figure out why is to have the robot send information about what it's doing to you. To do that, you can use the `telemetry.addData(String caption, Object value)` method. It's very similar to Java's `System.out.println()`. For example usage, look at some of the sample OpModes.

* `telemetry.addData(String caption, Object value)`
    * Adds an item to the end of the telemetry being built for driver station display.
* `telemetry.update()`
    * Sends the receiver Telemetry to the driver station

## Teleop

A Teleop is any OpMode which is controlled by the driver; it should run during the driver-controlled period of the match.

Any Teleop should have a looping section of code to be called repeatedly which reads the input and sets hardware parameters like motor powers or servo positions. This would be the inside of a `loop() {}` method in an `OpMode` or a `while (opModeIsActive()) {}` loop.

#### Controls

`OpMode` contains two `Gamepad` objects, `gamepad1` and `gamepad2`. These serve as your access point to the controllers. You can expect the FTC systems to actively update the values within these objects, so using them is as simple as something like `gamepad1.a` to check if the 'a' button on the first gamepad is pressed. To see a complete list, use the editor's autocomplete.

#### Tank and POV controls

Both of these driving methods are demonstrated in the `BasicOpMode_Iterative` and `BasicOpMode_Linear` samples. Reference the sample code to see how it can be done.

The simplest way to drive a robot is to directly set the left and right joystick y-values as the respective motor powers. This is referred to as tank drive and can be done with the following code assuming the motors directions are configured to work with this. POV Mode uses left stick to go forward, and right stick to turn. This uses basic math to combine motions and is easier to drive straight.

#### Mecanum
    TODO

#### Controlling Mechanisms
    TODO

#### Field Centric Mecanum
    TODO

## Autonomous

An Autonomous program is an OpMode meant to operate on its own without a driver. It is meant for the thirty-second autonomous period of the match, so any Autonomous will have a thirty-second timer in the interface. Typically, this is done using a `LinearOpMode`.

#### Timed driving

The simplest way to have an autonomous which does _something_ is to just execute steps based on guessing about how long it will take. This method is definitively bad, but can be usable if hardware is consistent and you don't have a lot of time to create an autonomous.

For example, let's suppose we want to:
* drive forward
* turn a bit
* drive forward

```java
leftDrive.setPower(1);
rightDrive.setPower(1);
sleep(500);

leftDrive.setPower(-0.5);
rightDrive.setPower(0.5);
sleep(500);

leftDrive.setPower(1);
rightDrive.setPower(1);
sleep(500);
```

`PushbotAutoDriveByTime_Linear` is an example which uses this in an actual program.

Some of the biggest problems with this are:
1. Distance driven and angle turned will be inconsistent.
2. If you want it to drive a certain distance or turn a certain amount, the only way is to guess and check.

#### Encoder Driving

To address the first problem, we can use the motor's built-in encoders to measure the distance they travel and stop at a certain point. 

The simple approach here is something like this:

```java
// start the motors
leftDrive.setPower(1);
rightDrive.setPower(1);

// wait until the motors have moved a certain number of ticks
int start = leftDrive.getCurrentPosition()
while(Math.abs(leftDrive.getCurrentPosition() - start) < 2000) {
    idle();
}

// stop the motors.
leftDrive.setPower(0);
rightDrive.setPower(0);
```

This approach works well enough, but is a little choppy and still a bit inconsistent. A better approach is to use the motor's built-in `RUN_TO_POSITION` mode. This mode allows you to set a target position, and the motor will run to it smoothly and accurately using PID control.

```java
int ticks = 2000;
double speed = 0.5

int target = motor.getCurrentPosition() + ticks;

motor.setTargetPosition(target);
motor.setMode(DcMotor.RunMode.RUN_TO_POSITION);
motor.setPower(speed);

while (motor.isBusy) {
    idle();
}

motor.setPower(0);
motor.setMode(DcMotor.RunMode.RUN_USING_ENCODER);
```

`PushbotAutoDriveByEncoder_Linear` is an example which uses this in an actual program.



