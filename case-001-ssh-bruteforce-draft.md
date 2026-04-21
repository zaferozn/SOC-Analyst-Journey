# Case 001 - SSH Brute Force Draft

## Scenario
This case is based on a controlled brute-force style lab performed in my practice environment.

## What Happened
Repeated SSH login attempts were generated against a target system in the lab.

## What I Observed
- Multiple login attempts were visible during the exercise.
- Authentication-related logs were reviewed.
- Repeated activity from the same source was part of the pattern.
- A custom rule concept was also tested for repeated attempts.

## Why It Matters
This type of behavior may indicate a brute-force attempt against SSH services.

## Initial Analyst Thought
The first step is to verify whether the attempts were successful or failed, identify the source IP, review the targeted account, and determine whether the pattern meets alerting thresholds.

## What I Need to Improve
I still need to improve my ability to review the logs independently and explain the findings with more confidence.
