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

## Workflow Reconstruction
1. A Linux lab environment was accessed through SSH.
2. Two terminal sessions were used during the exercise.
3. One side was used to generate repeated login attempts in a controlled way.
4. The other side was used to observe alerts and log-related output.
5. The initial Wazuh output included raw data that was not yet fully clear to me.
6. The main focus remained on alert visibility and repeated authentication activity.
7. A custom threshold idea was used to generate a higher-severity alert after repeated attempts.

## What the Exercise Taught Me
- repeated failed login attempts can create visible alert patterns
- alert visibility is easier to understand than full raw log output at the beginning
- threshold-based logic can help identify repeated suspicious behavior
- I still need deeper understanding of rule structure and raw log interpretation
