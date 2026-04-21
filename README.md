# Wombat-CC Drivetrain

Wombat-CC Drivetrain is a C++ controller for a 4-motor holonomic or mecanum base on KIPR Wombat.

This library provides grouped movement APIs for:

- Encoder-based drive and strafe movement
- Line-tracking drive and strafe movement
- Rotate and diagonal primitives
- Line acquisition and line alignment helpers
- Runtime debug logging

## Contents

- Setup and include
- Motor and sensor mapping
- Initialization flow
- Two-sensor and four-sensor examples
- Behavior notes and tuning
- Full API reference in API.md

## Setup

Add the dependency to your consumer build.zig.zon.

```zig
.dependencies = .{
    .wombat_cc_lib_drivetrain = .{ .path = "../Wombat-CC-Drivetrain/" },
};
```

Include the header in C++ code.

```cpp
#include <Wombat-CC/Drivetrain.hpp>
```

## Motor And Sensor Mapping

Constructor parameters map to motors in this order:

- FL: front-left motor
- FR: front-right motor
- RL: rear-left motor
- RR: rear-right motor

Line sensors are configured separately after construction.

- FL_IR: front-left line sensor
- FR_IR: front-right line sensor
- RR_IR: rear-right line sensor
- RL_IR: rear-left line sensor

## Required Initialization Order

1. Construct drivetrain with motor ports.
2. Optionally set debug mode.
3. Set motor performance multipliers.
4. Configure line sensor ports, choosing either 2-sensor or 4-sensor mode.
5. Set line thresholds using the matching 2-sensor or 4-sensor overload.

Line-tracking calls are guarded and will print a warning and return if configuration is incomplete or mismatched.

## Quick Start: Two Sensors

```cpp
#include <Wombat-CC/Drivetrain.hpp>

int main()
{
    Drivetrain drivetrain(0, 1, 2, 3);

    drivetrain.SetDebugEnabled(true);
    drivetrain.SetPerformance(1.0, 1.0, 1.0, 1.0);

    drivetrain.ConfigureLineTrackingSensors(0, 1);
    drivetrain.SetLineTrackingThresholds(200, 200, 3600, 3600);

    drivetrain.DriveByEncoder.Forward(300, 800);
    drivetrain.DriveLineTracking.Forward(1000, 700);
    drivetrain.StrafeLineTracking.ToLineRight(600);

    return 0;
}
```

## Quick Start: Four Sensors

```cpp
#include <Wombat-CC/Drivetrain.hpp>

int main()
{
    Drivetrain drivetrain(0, 1, 2, 3);

    drivetrain.SetDebugEnabled(true);
    drivetrain.SetPerformance(1.0, 1.0, 1.0, 1.0);

    drivetrain.ConfigureLineTrackingSensors(0, 1, 2, 3);
    drivetrain.SetLineTrackingThresholds(
        200, 200, 200, 200,
        3600, 3600, 3600, 3600);

    drivetrain.DriveLineTracking.Forward(1200, 700);
    drivetrain.StrafeLineTracking.LeftOnToLine(500);
    drivetrain.Line.Square(500);

    return 0;
}
```

## Direction Conventions

- Drive: positive speed means forward.
- Strafe: positive speed means right.
- Rotate: positive speed means clockwise.
- Diagonal wrappers encode sign combinations internally.

## Four-Sensor Tracking Behavior

When four sensors are configured:

- Drive line tracking uses FL, FR, RL, and RR directly for per-wheel correction.
- Strafe line tracking uses FL, FR, RL, and RR directly for per-wheel correction.
- Left and right side checks are derived from corner sensors:
left = FL or RL, right = FR or RR.
- LeftOnToLine and RightOnToLine wait until all configured sensors have observed the line.

When only two sensors are configured, existing two-sensor behavior is preserved.

## Tuning Notes

- Thresholds are midpoint values between white and black calibration readings.
- Per-motor performance multipliers should be tuned before fine line-tracking tuning.
- If the robot oscillates, reduce speed and consider lowering correction aggressiveness in code.
- If one corner drifts, calibrate that sensor again and recheck motor multiplier values.

## Debug Logging

Enable debug logs either in code or by environment variable.

- SetDebugEnabled(true)
- WOMBAT_CC_DEBUG=1

Truthy values for WOMBAT_CC_DEBUG are:

- 1
- true
- TRUE
- on
- ON

## API Reference

Use API.md for full method-by-method reference:

- API.md
