📘 EV Telemetry Data Analytics Project – README
📌 Project Overview

This project analyzes real-world electric vehicle telemetry data to detect ignition events, charging events, and battery behavior patterns. The dataset represents noisy IoT telematics data with irregular timestamps, missing values, and sensor glitches, similar to production fleet data.

The goal is to build robust data pipelines and event detection logic that works under real-world data quality issues.

📂 Dataset Description

The project uses three main datasets:

1. Telemetry Data (telemetry_data.csv)

Contains continuous sensor readings such as:

Vehicle speed

Battery level

Odometer

Ignition status

Timestamped telemetry records

2. Trigger Data (triggers_soc.csv)

Contains event-based signals from vehicle firmware, including:

Ignition cylinder events (IGN_CYL)

Charging lifecycle events (EV_CHARGE_STATE)

Battery charge state values

3. Vehicle Mapping (vehicle_pnid_mapping.csv)

Maps device identifiers (PNID) to vehicle IDs to enable joining trigger data with telemetry.

✅ Task 1: Data Sanity and Quality Checks
Timestamp Validation

What was done:

Converted timestamps to datetime format

Sorted data by vehicle and time

Checked for backward timestamps and irregular gaps

Findings:

Irregular time gaps

Missing and backward timestamps

Inference:
IoT devices buffer data, lose connectivity, or reset clocks in real-world deployments.

Unit and Range Validation

Checked metrics:

Speed

Battery level

Odometer

Findings:

Speed spikes

Battery jumps

Odometer resets

Inference:
Sensor glitches and firmware resets occur in real telematics data.

Missing Data Analysis

Findings:

ID, vehicle ID, and timestamp are always present

Battery, speed, and odometer often missing

Inference:
Telemetry is event-based, not continuous streaming.

Vehicle ID Mapping Validation

Verified all telemetry vehicle IDs exist in mapping file

No orphan devices found

Inference:
Dataset is curated and join-ready.

Synthetic Ignition Events

Artificial ignition OFF events were added to simulate missing real-world signals and validate trip segmentation logic.

✅ Task 2: Ignition Event Extraction
Objective

Create a unified IgnitionEvents table:

vehicle_id | event | event_ts


Where event ∈ { ignitionOn, ignitionOff }

Data Sources Used
Telemetry Ignition

Used IGNITION_STATUS column

Converted on → ignitionOn, off → ignitionOff

Trigger Ignition Logs

Used IGN_CYL signals from trigger data

Converted ON → ignitionOn, OFF → ignitionOff

Mapped PNID → vehicle_id using mapping file

Synthetic Ignition Events

Added artificial OFF events to handle missing signals

Final Output

All ignition sources were merged and sorted chronologically into a unified ignition event table.

✅ Task 3: Charging Lifecycle Event Extraction
Objective

Extract charging lifecycle events with schema:

vehicle_id | event | event_ts


Where event ∈ { Active, Abort, Completed }

Methodology
Filtering Charging Signals

Selected rows where NAME = EV_CHARGE_STATE

Event Selection

Only:

Active

Abort

Completed

Other signals (IGN_CYL, CHARGE_STATE) were ignored.

PNID to Vehicle Mapping

Parsed PNID lists from mapping file

Exploded mapping to one-to-many relationships

Joined trigger data with vehicle mapping

Handling Missing Vehicle IDs

Some PNIDs had no mapping → vehicle_id = NaN

Inference:

Unregistered devices

Onboarding gaps

Data quality issues

These records were flagged or excluded.

✅ Task 4: Battery Association with Events
Method

Matched each event with nearest battery telemetry within ±300 seconds

If two readings were equally close, earlier one was chosen

If no reading found → marked as UNKNOWN

Findings

Many events returned UNKNOWN

Telemetry timestamps misaligned with event timestamps

Vehicle IDs mismatched across datasets

Inference:
Real telematics data contains gaps and synchronization issues.

⚡ Final Task: Charging Detection Logic
Charging Detection Strategy
1. Battery Smoothing

Battery values were smoothed to reduce noise and spikes.

2. Trend-Based Detection

Charging was detected using continuous increasing battery trend, not fixed thresholds.

3. Charging Start Rule

Charging starts when battery increases for multiple consecutive timestamps.
This detects both:

Slow household charging

Fast public charging

4. Fluctuation Handling

Small temporary decreases were ignored.
One or two decreases did not terminate a charging event.

5. Charging End Rule

Charging ends only when battery decreases continuously for several points, indicating unplugging or driving.

6. Noise Filtering

Very small battery gains were ignored to avoid counting sensor noise as charging.

7. Output

For each charging session:

Vehicle ID

Start timestamp

End timestamp

Total battery percentage gained

📊 Key Insights

Telemetry data is noisy, sparse, and irregular

SOC jumps indicate sparse sampling, not instantaneous charging

Fixed thresholds are unreliable for real-world charging detection

Trend-based state machine logic is robust and industry-aligned

🚀 Applications

Fleet monitoring

Driver behavior analysis

Charging infrastructure planning

Battery health analytics

✅ Final Conclusion

This project demonstrates real-world telematics data challenges and implements robust event detection pipelines. Trend-based charging detection and noise-aware event segmentation provide reliable analytics under sparse and noisy telemetry conditions, similar to production fleet systems.
