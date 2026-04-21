# Case 001 - SSH Brute Force Draft

## Scenario
This case is based on a controlled SSH brute-force simulation performed in a lab environment.

## What Happened
Repeated login attempts were generated against a Linux system through SSH in order to observe how the activity appeared in Wazuh.

## What I Observed
- Repeated authentication attempts were visible during the exercise.
- The initial Wazuh view included a large amount of raw log data.
- The main focus was on alerts rather than full raw-log interpretation.
- A custom threshold-based rule concept was tested to trigger a high-severity alert after repeated attempts.

## Why It Matters
Repeated SSH login attempts may indicate brute-force behavior and require investigation to determine whether access was gained.

## Initial Analyst Thought
The first step is to verify the source IP, review the targeted account, confirm whether the attempts failed or succeeded, and check whether the behavior matches brute-force thresholds.

## Current Limitation
At this stage, I have observed the workflow in practice, but I still need to improve my understanding of rule logic, OSSEC structure, and independent log analysis.
