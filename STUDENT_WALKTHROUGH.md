# 🤖 Making the Robot Move: The Student Guide

Welcome to the robot code! This project uses the **Coordinate Logic** style (Inline Command-Based), which is designed to be readable and logical.

Think of code like a human body:
*   **Subsystems** (`CANFuelSubsystem`, `CANDriveSubsystem`) are the **muscles and senses** (Motors, Encoders).
*   **Commands** are the **actions** (Run forward, stop, shoot).
*   **RobotContainer** is the **brain** (Connecting controller buttons to actions).

---

## 📂 The Map: Where do I code?

You will spend 90% of your time in these three files:

### 1. `subsystems/` (The Capabilities)
*   **Where:** `src/main/java/frc/robot/subsystems/`
*   **What:** Defines **what the hardware CAN do**.
*   **Your Job:**
    *   Initialize motors (SparkMax, TalonFX).
    *   Create simple methods like `public void intake()`.
    *   **Student Translation:** "I am teaching the arm how to move up."

### 2. `RobotContainer.java` (The Controls)
*   **Where:** `src/main/java/frc/robot/`
*   **What:** Connects the **controller** to the **subsystems**.
*   **Your Job:**
    *   Define your Joysticks/Controllers.
    *   Bind buttons: `controller.a().whileTrue(...)`
    *   **Student Translation:** "When I press 'A', do the 'Shoot' action."

### 3. `Constants.java` (The Magic Numbers)
*   **Where:** `src/main/java/frc/robot/`
*   **What:** A single place for all your tuning numbers.
*   **Your Job:** Speed values, PID gains, Controller ports.
*   **Student Translation:** "The arm is too fast, let me change `ARM_SPEED` from 1.0 to 0.5."

---

## 🧠 Step-by-Step Flow

Here is how a signal travels from your finger to the motor, using the actual **Fuel Subsystem** code.

### Step 1: The Subsystem (`CANFuelSubsystem.java`)
First, we separate the hardware from the logic. The subsystem owns the motor.

```java
public class CANFuelSubsystem extends SubsystemBase {
    // 1. Define the hardware
    private final SparkMax feederRoller;
    private final SparkMax intakeLauncherRoller;

    public CANFuelSubsystem() {
        // ... Config code hidden for simplicity ...
    }

    // 2. Define the basic ability (The Muscle)
    public void intake() {
        feederRoller.setVoltage(INTAKING_FEEDER_VOLTAGE);
        intakeLauncherRoller.setVoltage(INTAKING_INTAKE_VOLTAGE);
    }
    
    public void stop() {
        feederRoller.set(0);
        intakeLauncherRoller.set(0);
    }

    // 3. Define a Command Factory (Optional but clean)
    // This creates a command that runs the spinUp method forever
    public Command spinUpCommand() {
        return this.run(() -> spinUp());
    }
}
```

### Step 2: The Binding (`RobotContainer.java`)
Now we tell the robot *when* to use that ability.

```java
public class RobotContainer {
    // 1. Create the specific subsystem
    private final CANFuelSubsystem ballSubsystem = new CANFuelSubsystem();
    
    // 2. Create the controller
    private final CommandXboxController operatorController = new CommandXboxController(OPERATOR_CONTROLLER_PORT);

    private void configureBindings() {
        // 3. Bind the Button -> to the Action
        
        // SIMPLE: "While holding Left Bumper, run intake. When released, stop."
        operatorController.leftBumper()
            .whileTrue(ballSubsystem.runEnd(
                () -> ballSubsystem.intake(),  // Run this while held
                () -> ballSubsystem.stop()     // Run this when released
            ));

        // COMPLEX: "Spin up for 2 seconds, then Shoot, then Stop automatically"
        operatorController.rightBumper()
            .whileTrue(
                ballSubsystem.spinUpCommand()         // 1. Start spinning
                .withTimeout(2.0)                     // 2. Wait 2 seconds
                .andThen(ballSubsystem.launchCommand()) // 3. Then Shoot
                .finallyDo(() -> ballSubsystem.stop()) // 4. Always stop at the end
            );
    }
}
```

---

## ⚡ Cheat Sheet: The "Inline" Verbs

In `RobotContainer`, you will often chain these "decorators" to make complex things happen easily.

| Code | Plain English |
| :--- | :--- |
| `.onTrue(cmd)` | "When I **tap** the button, start this command." |
| `.whileTrue(cmd)` | "Run this **as long as** I hold the button." |
| `.andThen(nextCmd)` | "Finish the first thing, **and then** do the next thing." |
| `.withTimeout(3.0)` | "Run this, but **force stop** it after 3 seconds." |
| `.runOnce(() -> ...)`| "Do this one thing **instantly**, then finish." |
| `.runEnd(start, end)`| "Run `start` while active, run `end` when finished." |
| `.finallyDo(() -> ...)` | "No matter what happens (finish or cancel), do this last." |

---

## 🛑 Common Pitfalls (Watch out!)

1.  **Ghost Commands:** If you create a command but don't bind it to a button or put it in `setDefaultCommand`, it will never run!
2.  **The "Requirement" Rule:** A subsystem can only do **one thing at a time**.
    *   If the robot is `Driving`, and you press a button to `Auto-Align` (which also uses the Drive Subsystem), the `Driving` command gets cancelled immediately.
3.  **Port Collisions:** If `Constants.java` has the same ID for the Intake Motor and the Drive Motor, your robot will seize up or crash. Always check your IDs!
4.  **Voltage vs Speed:** `.set(0.5)` is 50% duty cycle. `.setVoltage(6.0)` is 6 volts (out of ~12V). They are similar but not identical!

---

## 🚀 How to Deploy

1.  **Connect to Robot:** Connect your laptop to the robot via USB or Radio.
2.  **Build Code:** Click the **WPILib** icon (the W in the corner) -> **Build Robot Code**.
3.  **Deploy:** Click **WPILib** -> **Deploy Robot Code**.
4.  **Enable:** Open the **Driver Station**, check that you have "Communication" and "Robot Code", then click **Enable**.
