# Drivetrain API Reference

This document is the complete API reference for the Drivetrain class in Wombat-CC-Drivetrain.

## Class Construction

Signature:

```cpp
Drivetrain(int FL_motor, int FR_motor, int RL_motor, int RR_motor)
```

Parameters:

- FL_motor: Front-left motor port
- FR_motor: Front-right motor port
- RL_motor: Rear-left motor port
- RR_motor: Rear-right motor port

Notes:

- Constructor only configures motors.
- Line sensors are configured separately through ConfigureLineTrackingSensors overloads.

## Configuration Methods

### ConfigureLineTrackingSensors, Two Sensors

```cpp
void ConfigureLineTrackingSensors(int FL_IR, int FR_IR)
```

Use this overload for a front sensor pair.

### ConfigureLineTrackingSensors, Four Sensors

```cpp
void ConfigureLineTrackingSensors(int FL_IR, int FR_IR, int RR_IR, int RL_IR)
```

Use this overload when all four corners have line sensors.

### SetLineTrackingThresholds, Two Sensors

```cpp
void SetLineTrackingThresholds(int FL_white, int FR_white, int FL_black, int FR_black)
```

### SetLineTrackingThresholds, Four Sensors

```cpp
void SetLineTrackingThresholds(
    int FL_white,
    int FR_white,
    int RR_white,
    int RL_white,
    int FL_black,
    int FR_black,
    int RR_black,
    int RL_black)
```

Rules:

- Sensor count and threshold count must match.
- Two-sensor ports require two-sensor thresholds.
- Four-sensor ports require four-sensor thresholds.

### SetPerformance

```cpp
void SetPerformance(double FLP, double FRP, double RLP, double RRP)
```

Purpose:

- Apply per-motor compensation multipliers.

### Debug Methods

```cpp
void SetDebugEnabled(bool enabled)
bool IsDebugEnabled() const
```

### Configuration Status

```cpp
bool IsLineTrackingConfigured() const
```

Returns true only when:

- Sensor ports are configured
- Thresholds are configured
- Sensor and threshold mode match, both 2 or both 4

## Movement API Groups

All movement calls are grouped under public members:

- DriveByEncoder
- DriveLineTracking
- StrafeByEncoder
- StrafeLineTracking
- Rotate
- Diagonal
- Line

## DriveByEncoder

```cpp
drivetrain.DriveByEncoder.Forward(int ticks, int speed)
drivetrain.DriveByEncoder.Backward(int ticks, int speed)
```

Behavior:

- Uses encoder target ticks.
- Uses configured performance multipliers.

## DriveLineTracking

```cpp
drivetrain.DriveLineTracking.Forward(int ticks, int speed)
drivetrain.DriveLineTracking.Backward(int ticks, int speed)
drivetrain.DriveLineTracking.ForwardToLine(int speed)
drivetrain.DriveLineTracking.BackwardToLine(int speed)
```

Behavior:

- Requires valid line-tracking configuration.
- Four-sensor mode uses FL FR RR RL directly in correction loops.
- Two-sensor mode preserves legacy front-pair behavior.

## StrafeByEncoder

```cpp
drivetrain.StrafeByEncoder.Left(int ticks, int speed)
drivetrain.StrafeByEncoder.Right(int ticks, int speed)
```

## StrafeLineTracking

```cpp
drivetrain.StrafeLineTracking.Left(int ticks, int speed)
drivetrain.StrafeLineTracking.Right(int ticks, int speed)

drivetrain.StrafeLineTracking.LeftToLine(int speed)
drivetrain.StrafeLineTracking.RightToLine(int speed)

drivetrain.StrafeLineTracking.ToLineLeft(int speed)
drivetrain.StrafeLineTracking.ToLineRight(int speed)

drivetrain.StrafeLineTracking.LeftOnToLine(int speed)
drivetrain.StrafeLineTracking.RightOnToLine(int speed)
```

Notes:

- LeftToLine and ToLineLeft are equivalent aliases.
- RightToLine and ToLineRight are equivalent aliases.
- LeftOnToLine and RightOnToLine require all configured sensors to detect the line in four-sensor mode.

## Rotate

```cpp
drivetrain.Rotate.Left(int ticks, int speed)
drivetrain.Rotate.Right(int ticks, int speed)
```

## Diagonal

```cpp
drivetrain.Diagonal.ForwardLeft(int ticks, int speed)
drivetrain.Diagonal.ForwardRight(int ticks, int speed)
drivetrain.Diagonal.BackwardLeft(int ticks, int speed)
drivetrain.Diagonal.BackwardRight(int ticks, int speed)
```

## Line

```cpp
drivetrain.Line.Square(int speed)
drivetrain.Line.Center(int speed)
```

Behavior:

- Square attempts to align both sides to a line.
- Center attempts to center off line state according to current correction rules.

## Direction Conventions

- Drive: positive speed means forward
- Strafe: positive speed means right
- Rotate: positive speed means clockwise

## Error And Warning Behavior

Line-tracking methods print a warning and return if configuration is invalid.

Typical causes:

- Sensors not configured
- Thresholds not configured
- Configured sensor count does not match threshold count

## Recommended Startup Sequence

```cpp
Drivetrain drivetrain(FL, FR, RL, RR);
drivetrain.SetPerformance(1.0, 1.0, 1.0, 1.0);

drivetrain.ConfigureLineTrackingSensors(FL_IR, FR_IR, RR_IR, RL_IR);
drivetrain.SetLineTrackingThresholds(
    FL_white, FR_white, RR_white, RL_white,
    FL_black, FR_black, RR_black, RL_black);
```

Then call grouped movement methods as needed.
