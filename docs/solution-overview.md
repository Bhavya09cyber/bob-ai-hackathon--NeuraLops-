# Solution Overview

## What We Built

[We built **CareSentinel**, an AI-powered healthcare care-gap detection and monitoring system.

CareSentinel creates a living timeline of a patient's healthcare journey and connects events such as consultations, tests, reports, referrals, and follow-ups.

Instead of only reminding users about known tasks, the system checks whether the expected sequence of care is actually happening.

When an expected step is missing or delayed, CareSentinel identifies the potential care gap and provides an explainable alert and suggest what to do next . ]

## How It Works

[1. Healthcare events such as consultations, tests, reports, referrals, and follow-ups are recorded for a patient.

2. CareSentinel organizes these events into a chronological care journey.

3. The system compares the recorded events with expected relationships between care-process steps.

4. If an expected event does not occur within the defined time window, the care-gap detection engine identifies a potential gap.

5. The system generates a priority-based alert for the detected care gap.

6. IBM watsonx.ai with IBM Granite can analyze the detected gap and generate an understandable summary, explanation, confidence information, and suggested next care-process action.

7. The care team or caregiver can review the patient's journey, detected gaps, and AI-generated insights through the dashboard.

8. Healthcare documents can also be uploaded and processed to extract relevant care events and add them to the patient's journey.]

## Architecture Diagram

> See [`architecture.md`](architecture.md) for the detailed diagram.

[Care Team / User]
        |
        v
[Frontend: HTML/CSS/JavaScript]
        |
        | REST API
        v
[Flask Backend]
        |
        +--------------------+
        |                    |
        v                    v
[SQLite Database]     [Document Processor]
        |                    |
        |                    v
        |             [Event Extraction]
        |                    |
        +---------+----------+
                  |
                  v
        [Care-Gap Detector]
                  |
                  v
        [IBM watsonx.ai]
        [IBM Granite]
                  |
                  v
        [Explainable Alerts
         + Journey Insights]
```

## Key Design Decisions

Decision	Rationale
Used a living care journey timeline	- Allows connected healthcare events to be viewed as one continuous process instead of isolated tasks.
Used expected vs. actual care comparison	- Helps identify missing or delayed steps in the care journey.
Used rule-based care-gap detection	Provides deterministic and explainable detection of expected care-process relationships.
Used IBM watsonx.ai with IBM Granite	- Provides AI-powered explanation, classification, summarization, and journey insights for detected care gaps.
Used SQLite	- Provides a lightweight database suitable for the hackathon prototype and local demonstration.
Added document processing	- Allows healthcare documents to contribute relevant events to the patient's care journey.
Added patient-specific monitoring	- Ensures that events and alerts are associated with the correct patient's care journey.
## IBM Technologies Used

IBM watsonx.ai: Used as the runtime AI service for analyzing detected care gaps and generating explainable care-process insights when valid watsonx.ai credentials are configured.

IBM Granite: Used through IBM watsonx.ai to analyze care-gap information and generate structured AI insights such as summaries, explanations, confidence information, and suggested next care-process actions.

IBM Bob: Used as the AI-powered development assistant during the project for code generation, debugging, refactoring, documentation, and development support. IBM Bob is a development tool and is not the runtime AI engine of the CareSentinel application
